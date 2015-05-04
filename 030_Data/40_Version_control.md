## 处理冲突

当使用`index` API更新文档的时候，我们读取原始文档，做修改，然后将**整个文档(whole document)**一次性重新索引。最近的索引请求会生效——Elasticsearch中只存储最后被索引的任何文档。如果其他人同时也修改了这个文档，他们的修改将会丢失。

很多时候，这并不是一个问题。或许我们主要的数据存储在关系型数据库中，然后拷贝数据到Elasticsearch中只是为了可以用于搜索。或许两个人同时修改文档的机会很少。亦或者偶尔的修改丢失对于我们的工作来说并无大碍。

但有时丢失修改是一个**很严重**的问题。想象一下我们使用Elasticsearch存储大量在线商店的库存信息。每当销售一个商品，Elasticsearch中的库存就要减一。

一天，老板决定做一个促销。瞬间，我们每秒就销售了几个商品。想象两个同时运行的web进程，两者同时处理一件商品的订单：

![img-data-lww](https://raw.githubusercontent.com/looly/elasticsearch-definitive-guide-cn/master/images/elas_0301.png)

`web_1`让`stock_count`失效是因为`web_2`没有察觉到`stock_count`的拷贝已经过期（译者注：`web_1`取数据，减一后更新了`stock_count`。可惜在`web_1`更新`stock_count`前它就拿到了数据，这个数据已经是过期的了，当`web_2`再回来更新`stock_count`时这个数字就是错的。这样就会造成看似卖了一件东西，其实是卖了两件，这个应该属于幻读。）。结果是我们认为自己确实还有更多的商品，最终顾客会因为销售给他们没有的东西而失望。

变化越是频繁，或读取和更新间的时间越长，越容易丢失我们的更改。

在数据库中，有两种通用的方法确保在并发更新时修改不丢失：

### 悲观并发控制（Pessimistic concurrency control）

这在关系型数据库中被广泛的使用，假设冲突的更改经常发生，为了解决冲突我们把访问区块化。典型的例子是在读一行数据前锁定这行，然后确保只有加锁的那个线程可以修改这行数据。

### 乐观并发控制（Optimistic concurrency control）：

被Elasticsearch使用，假设冲突不经常发生，也不区块化访问，然而，如果在读写过程中数据发生了变化，更新操作将失败。这时候由程序决定在失败后如何解决冲突。实际情况中，可以重新尝试更新，刷新数据（重新读取）或者直接反馈给用户。

## 乐观并发控制

Elasticsearch是分布式的。当文档被创建、更新或删除，文档的新版本会被复制到集群的其它节点。Elasticsearch即是同步的又是异步的，意思是这些复制请求都是平行发送的，并**无序(out of sequence)**的到达目的地。这就需要一种方法确保老版本的文档永远不会覆盖新的版本。

上文我们提到`index`、`get`、`delete`请求时，我们指出每个文档都有一个`_version`号码，这个号码在文档被改变时加一。Elasticsearch使用这个`_version`保证所有修改都被正确排序。当一个旧版本出现在新版本之后，它会被简单的忽略。

我们利用`_version`的这一优点确保数据不会因为修改冲突而丢失。我们可以指定文档的`version`来做想要的更改。如果那个版本号不是现在的，我们的请求就失败了。

Let's create a new blog post:
让我们创建一个新的博文：

```Javascript
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```

响应体告诉我们这是一个新建的文档，它的`_version`是`1`。现在假设我们要编辑这个文档：把数据加载到web表单中，修改，然后保存成新版本。

首先我们检索文档：

```Javascript
GET /website/blog/1
```

响应体包含相同的`_version`是`1`

```Javascript
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}
```

现在，当我们通过重新索引文档保存修改时，我们这样指定了`version`参数：

```Javascript
PUT /website/blog/1?version=1 <1>
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```
- <1> 我们只希望文档的`_version`是`1`时更新才生效。

This request succeeds, and the response body tells us that the `_version`
has been incremented to `2`:

请求成功，响应体告诉我们`_version`已经增加到`2`：

```Javascript
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "1",
  "_version": 2
  "created":  false
}
```

然而，如果我们重新运行相同的索引请求，依旧指定`version=1`，Elasticsearch将返回`409 Conflict`状态的HTTP响应。响应体类似这样：

```Javascript
{
  "error" : "VersionConflictEngineException[[website][2] [blog][1]:
             version conflict, current [2], provided [1]]",
  "status" : 409
}
```

这告诉我们当前`_version`是`2`，但是我们指定想要更新的版本是`1`。

我们需要做什么取决于程序的需求。我们可以告知用户其他人修改了文档，你应该在保存前再看一下。而对于上文提到的商品`stock_count`，我们需要重新检索最新文档然后申请新的更改操作。

所有更新和删除文档的请求都接受`version`参数，它可以允许在你的代码中增加乐观锁控制。

## 使用外部版本控制系统

一种常见的结构是使用一些其他的数据库做为主数据库，然后使用Elasticsearch搜索数据，这意味着所有主数据库发生变化，就要将其拷贝到Elasticsearch中。如果有多个进程负责这些数据的同步，就会遇到上面提到的并发问题。

如果主数据库有版本字段——或一些类似于`timestamp`等可以用于版本控制的字段——是你就可以在Elasticsearch的查询字符串后面添加`version_type=external`来使用这些版本号。版本号必须是整数，大于零小于`9.2e+18`——Java中的正的`long`。

外部版本号与之前说的内部版本号在处理的时候有些不同。它不再检查`_version`是否与请求中指定的**一致**，而是检查是否**小于**指定的版本。如果请求成功，外部版本号就会被存储到`_version`中。

外部版本号不仅在索引和删除请求中指定，也可以在**创建(create)**新文档中指定。

例如，创建一个包含外部版本号`5`的新博客，我们可以这样做：

```Javascript
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```

在响应中，我们能看到当前的`_version`号码是`5`：

```Javascript
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 5,
  "created":  true
}
```

现在我们更新这个文档，指定一个新`version`号码为`10`：

```Javascript
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```

请求成功的设置了当前`_version`为`10`：

```Javascript
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 10,
  "created":  false
}
```

如果你重新运行这个请求，就会返回一个像之前一样的冲突错误，因为指定的外部版本号不大于当前在Elasticsearch中的版本。

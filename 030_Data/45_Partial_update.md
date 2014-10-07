## 文档局部更新

在《更新文档》一章，我们说了一种通过检索，修改，然后重建整文档的索引方法来更新文档。这是对的。然而，使用`update` API，我们可以使用一个请求来实现局部更新，例如增加数量的操作。

我们也说过文档是不可变的——它们不能被更改，只能被替换。`update` API**必须**遵循相同的规则。表面看来，我们似乎是局部更新了文档的位置，内部却是像我们之前说的一样简单的使用`update` API处理相同的*检索-修改-重建索引*流程，我们也减少了其他进程可能导致冲突的修改。

最简单的`update`请求表单接受一个局部文档参数`doc`，它会合并到现有文档中——对象合并在一起，存在的标量字段被覆盖，新字段被添加。举个例子，我们可以使用以下请求为博客添加一个`tags`字段和一个`views`字段：

```Javascript
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

如果请求成功，我们将看到类似`index`请求的响应结果：

```
{
   "_index" :   "website",
   "_id" :      "1",
   "_type" :    "blog",
   "_version" : 3
}
```

检索文档文档显示被更新的`_source`字段：

```Javascript
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  3,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags": [ "testing" ], <1>
      "views":  0 <1>
   }
}
```

- <1> 我们新添加的字段已经被添加到`_source`字段中。

### 使用脚本局部更新

> ### 使用Groovy脚本

> 这时候当API不能满足要求时，Elasticsearch允许你使用脚本实现自己的逻辑。脚本支持非常多的API，例如搜索、排序、聚合和文档更新。脚本可以通过请求的一部分、检索特殊的`.scripts`索引或者从磁盘加载方式执行。

> 默认的脚本语言是[Groovy](http://groovy.codehaus.org/)，一个快速且功能丰富的脚本语言，语法类似于Javascript。它在一个**沙盒(sandbox)**中运行，以防止恶意用户毁坏Elasticsearch或攻击服务器。

> 你可以在《脚本参考文档》中获得更多信息。

脚本能够使用`update` API改变`_source`字段的内容，它在脚本内部以`ctx._source`表示。例如，我们可以使用脚本增加博客的`views`数量：

```Javascript
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```

我们还可以使用脚本增加一个新标签到`tags`数组中。在这个例子中，我们定义了一个新标签做为参数而不是硬编码在脚本里。这允许Elasticsearch未来可以重复利用脚本，而不是在想要增加新标签时必须每次编译新脚本：

```Javascript
POST /website/blog/1/_update
{
   "script" : "ctx._source.tags+=new_tag",
   "params" : {
      "new_tag" : "search"
   }
}
```

获取最后两个有效请求的文档：

```Javascript
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  5,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags":  ["testing", "search"], <1>
      "views":  1 <2>
   }
}
```

- <1> `search`标签已经被添加到`tags`数组。
- <2> `views`字段已经被增加。

通过设置`ctx.op`为`delete`我们可以根据内容删除文档：

```Javascript
POST /website/blog/1/_update
{
   "script" : "ctx.op = ctx._source.views == count ? 'delete' : 'none'",
    "params" : {
        "count": 1
    }
}
```

### 更新可能不存在的文档

想象我们要在Elasticsearch中存储浏览量计数器。每当有用户访问页面，我们增加这个页面的浏览量。但如果这是个新页面，我们并不确定这个计数器存在与否。当我们试图更新一个不存在的文档，更新将失败。

在这种情况下，我们可以使用`upsert`参数定义文档来使其不存在时被创建。

```Javascrupt
POST /website/pageviews/1/_update
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
```

第一次执行这个请求，`upsert`值被索引为一个新文档，初始化`views`字段为`1`.接下来文档已经存在，所以`script`被更新代替，增加`views`数量。

### 更新和冲突

In the introduction to this section, we said that the smaller the window between
the _retrieve_ and _reindex_ steps, the smaller the opportunity for
conflicting changes. But it doesn't eliminate the possibility completely. It
is still possible that a request from another process could change the
document before `update` has managed to reindex it.

To avoid losing data, the `update` API retrieves the current `_version`
of the document in the _retrieve_ step, and passes that to the `index` request
during the _reindex_ step.
If another process has changed the document in between _retrieve_ and _reindex_,
then the `_version` number won't match and the update request will fail.

For many uses of partial update, it doesn't matter that a document has been
changed.  For instance, if two processes are both incrementing the page
view counter, it doesn't matter in which order it happens -- if a conflict
occurs, the only thing we need to do is to reattempt the update.

This can be done automatically by setting the `retry_on_conflict` parameter to
the number of times that `update` should retry before failing -- it defaults
to `0`.

[source,js]
--------------------------------------------------
POST /website/pageviews/1/_update?retry_on_conflict=5 <1>
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 0
   }
}
--------------------------------------------------
// SENSE: 030_Data/45_Upsert.json
<1> Retry this update 5 times before failing.

This works well for operations like incrementing a counter where the order of
increments does not matter, but there are other situations where the order of
changes *is* important. Like the <<index-doc,`index` API>>, the `update` API
adopts a ``last-write-wins'' approach by default, but it also accepts a
`version` parameter which allows you to use
<<optimistic-concurrency-control,optimistic concurrency control>> to specify
which version of the document you intend to update.


## 索引一个文档

文档通过`index` API被索引——使数据可以被存储和搜索。但是首先我们需要决定文档所在。正如我们讨论的，文档通过其`_index`、`_type`、`_id`唯一确定。们可以自己提供一个`_id`，或者也使用`index` API 为我们生成一个。

### 使用自己的ID

如果你的文档有自然的标识符（例如`user_account`字段或者其他值表示文档），你就可以提供自己的`_id`，使用这种形式的`index` API：

```Javascript
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

例如我们的索引叫做`“website”`，类型叫做`“blog”`，我们选择的ID是`“123”`，那么这个索引请求就像这样：

```Javascript
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}
```

Elasticsearch的响应：

```Javascript
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "123",
   "_version":  1,
   "created":   true
}
```

响应指出请求的索引已经被成功创建，这个索引中包含`_index`、`_type`和`_id`元数据，以及一个新元素：`_version`。

Elasticsearch中每个文档都有版本号，每当文档变化（包括删除）都会使`_version`增加。在《版本控制》章节中我们将探讨如何使用`_version`号确保你程序的一部分不会覆盖掉另一部分所做的更改。

### 自增ID

如果我们的数据没有自然ID，我们可以让Elasticsearch自动为我们生成。请求结构发生了变化：`PUT`方法——`“在这个URL中存储文档”`变成了`POST`方法——`"在这个类型下存储文档"`。（译者注：原来是把文档存储到某个ID对应的空间，现在是把这个文档添加到某个`_type`下）。

URL现在只包含`_index`和`_type`两个字段：

```Javascript
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

响应内容与刚才类似，只有`_id`字段变成了自动生成的值：

```Javascript
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "wM0OSFhDQXGZAWDf0-drSA",
   "_version":  1,
   "created":   true
}
```

自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally unique identifiers, 或者叫 [UUIDs](http://en.wikipedia.org/wiki/Uuid)。


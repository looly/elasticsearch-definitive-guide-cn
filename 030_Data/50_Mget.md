## 检索多个文档

像Elasticsearch一样，检索多个文档依旧非常快。合并多个请求可以避免每个请求单独的网络开销。如果你需要从Elasticsearch中检索多个文档，相对于一个一个的检索，更快的方式是在一个请求中使用**multi-get**或者`mget` API。

`mget` API参数是一个`docs`数组，数组的每个节点定义一个文档的`_index`、`_type`、`_id`元数据。如果你只想检索一个或几个确定的字段，也可以定义一个`_source`参数：

```Javascript
POST /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```

响应体也包含一个`docs`数组，每个文档还包含一个响应，它们按照请求定义的顺序排列。每个这样的响应与单独使用**`get` request**响应体相同：

```Javascript
{
   "docs" : [
      {
         "_index" :   "website",
         "_id" :      "2",
         "_type" :    "blog",
         "found" :    true,
         "_source" : {
            "text" :  "This is a piece of cake...",
            "title" : "My first external blog entry"
         },
         "_version" : 10
      },
      {
         "_index" :   "website",
         "_id" :      "1",
         "_type" :    "pageviews",
         "found" :    true,
         "_version" : 2,
         "_source" : {
            "views" : 2
         }
      }
   ]
}
```

如果你想检索的文档在同一个`_index`中（甚至在同一个`_type`中），你就可以在URL中定义一个默认的`/_index`或者`/_index/_type`。

你依旧可以在单独的请求中使用这些值：

```Javascript
POST /website/blog/_mget
{
   "docs" : [
      { "_id" : 2 },
      { "_type" : "pageviews", "_id" :   1 }
   ]
}
```

事实上，如果所有文档具有相同`_index`和`_type`，你可以通过简单的`ids`数组来代替完整的`docs`数组：

```Javascript
POST /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```

注意到我们请求的第二个文档并不存在。我们定义了类型为`blog`，但是ID为`1`的文档类型为`pageviews`。这个不存在的文档会在响应体中被告知。

```Javascript
{
  "docs" : [
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "2",
      "_version" : 10,
      "found" :    true,
      "_source" : {
        "title":   "My first external blog entry",
        "text":    "This is a piece of cake..."
      }
    },
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "1",
      "found" :    false  <1>
    }
  ]
}
```
- <1> 这个文档不存在

事实上第二个文档不存在并不影响第一个文档的检索。每个文档的检索和报告都是独立的。

> 注意：

> 尽管前面提到有一个文档没有被找到，但HTTP请求状态码还是`200`。事实上，就算所有文档都找不到，请求也还是返回`200`，原因是`mget`请求本身成功了。如果想知道每个文档是否都成功了，你需要检查`found`标志。

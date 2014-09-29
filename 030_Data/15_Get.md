## 检索文档

想要从Elasticsearch中获取文档，我们使用同样的`_index`、`_type`、`_id`，但是HTTP方法改为`GET`：

```Javascript
GET /website/blog/123?pretty
```
响应包含了现在熟悉的元数据节点，增加了`_source`字段，它包含了在创建索引时我们发送给Elasticsearch的原始文档。

```Javascript
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out...",
      "date":  "2014/01/01"
  }
}
```

> ### `pretty`

>在任意的查询字符串中增加`pretty`参数，类似于上面的例子。会让Elasticsearch**美化输出(pretty-print)**JSON响应以便更加容易阅读。`_source`字段不会被美化，它的样子与我们输入的一致。

GET请求返回的响应内容包括`{"found": true}`。这意味着文档已经找到。如果我们请求一个不存在的文档，依旧会得到一个JSON，不过`found`值变成了`false`。

此外，HTTP响应状态码也会变成`'404 Not Found'`代替`'200 OK'`。我们可以在`curl`后加`-i`参数得到响应头：

```sh
curl -i -XGET http://localhost:9200/website/blog/124?pretty
```

现在响应类似于这样：

```Javascript
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=UTF-8
Content-Length: 83

{
  "_index" : "website",
  "_type" :  "blog",
  "_id" :    "124",
  "found" :  false
}
```

### 检索文档的一部分

通常，`GET`请求将返回文档的全部，存储在`_source`参数中。但是可能你感兴趣的字段只是`title`。请求个别字段可以使用`_source`参数。多个字段可以使用逗号分隔：

```sh
GET /website/blog/123?_source=title,text
```

`_source`字段现在只包含我们请求的字段，而且过滤了`date`字段：

```Javascript
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "exists" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```

或者你只想得到`_source`字段而不要其他的元数据，你可以这样请求：

```sh
GET /website/blog/123/_source
```
它仅仅返回:

```Javascript
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```

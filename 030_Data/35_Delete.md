## 删除文档

删除文档的语法模式与之前基本一致，只不过要使用`DELETE`方法：

```Javascript
DELETE /website/blog/123
```

如果文档被找到，Elasticsearch将返回`200 OK`状态码和以下响应体。注意`_version`数字已经增加了。

```Javascript
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```

如果文档未找到，我们将得到一个`404 Not Found`状态码，响应体是这样的：

```Javascript
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```

尽管文档不存在——`"found"`的值是`false`——`_version`依旧增加了。这是内部记录的一部分，它确保在多节点间不同操作可以有正确的顺序。

> 正如在《更新文档》一章中提到的，删除一个文档也不会立即从磁盘上移除，它只是被标记成已删除。Elasticsearch将会在你之后添加更多索引的时候才会在后台进行删除内容的清理。


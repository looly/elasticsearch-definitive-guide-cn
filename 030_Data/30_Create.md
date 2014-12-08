## 创建一个新文档

当索引一个文档，我们如何确定是完全创建了一个新的还是覆盖了一个已经存在的呢？

请记住`_index`、`_type`、`_id`三者唯一确定一个文档。所以要想保证文档是新加入的，最简单的方式是使用`POST`方法让Elasticsearch自动生成唯一`_id`：

```Javascript
POST /website/blog/
{ ... }
```

然而，如果想使用自定义的`_id`，我们必须告诉Elasticsearch应该在`_index`、`_type`、`_id`三者都不同时才接受请求。为了做到这点有两种方法，它们其实做的是同一件事情。你可以选择适合自己的方式：

第一种方法使用`op_type`查询参数：

```Javascript
PUT /website/blog/123?op_type=create
{ ... }
```

或者第二种方法是在URL后加`/_create`做为端点：

```Javascript
PUT /website/blog/123/_create
{ ... }
```

如果请求成功的创建了一个新文档，Elasticsearch将返回正常的元数据且响应状态码是`201 Created`。

另一方面，如果包含相同的`_index`、`_type`和`_id`的文档已经存在，Elasticsearch将返回`409 Conflict`响应状态码，错误信息类似如下：

```Javascript
{
  "error" : "DocumentAlreadyExistsException[[website][4] [blog][123]:
             document already exists]",
  "status" : 409
}
```

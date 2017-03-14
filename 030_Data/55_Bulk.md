## 更新时的批量操作

就像`mget`允许我们一次性检索多个文档一样，`bulk` API允许我们使用单一请求来实现多个文档的`create`、`index`、`update`或`delete`。这对索引类似于日志活动这样的数据流非常有用，它们可以以成百上千的数据为一个批次按序进行索引。

`bulk`请求体如下，它有一点不同寻常：

```Javascript
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```

这种格式类似于用`"\n"`符号连接起来的一行一行的JSON文档**流(stream)**。两个重要的点需要注意：

* 每行必须以`"\n"`符号结尾，**包括最后一行**。这些都是作为每行有效的分离而做的标记。

* 每一行的数据不能包含未被转义的换行符，它们会干扰分析——这意味着JSON不能被美化打印。

> 提示:

> 在《批量格式》一章我们介绍了为什么`bulk` API使用这种格式。

**action/metadata**这一行定义了**文档行为(what action)**发生在**哪个文档(which document)**之上。

**行为(action)**必须是以下几种：

| 行为     | 解释                                                   |
| -------- | ------------------------------------------------------ |
| `create` | 当文档不存在时创建之。详见《创建文档》                 |
| `index`  | 创建新文档或替换已有文档。见《索引文档》和《更新文档》 |
| `update` | 局部更新文档。见《局部更新》                           |
| `delete` | 删除一个文档。见《删除文档》                           |


在索引、创建、更新或删除时必须指定文档的`_index`、`_type`、`_id`这些**元数据(metadata)**。

例如删除请求看起来像这样：

```Javascript
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
```

**请求体(request body)**由文档的`_source`组成——文档所包含的一些字段以及其值。它被`index`和`create`操作所必须，这是有道理的：你必须提供文档用来索引。

这些还被`update`操作所必需，而且请求体的组成应该与`update` API（`doc`, `upsert`,
`script`等等）一致。删除操作不需要**请求体(request body)**。

```Javascript
{ "create":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
```

如果定义`_id`，ID将会被自动创建：

```Javascript
{ "index": { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
```

为了将这些放在一起，`bulk`请求表单是这样的：

```Javascript
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} <1>
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } <2>

```

- <1> 注意`delete`**行为(action)**没有请求体，它紧接着另一个**行为(action)**
- <2> 记得最后一个换行符

Elasticsearch响应包含一个`items`数组，它罗列了每一个请求的结果，结果的顺序与我们请求的顺序相同：

```Javascript
{
   "took": 4,
   "errors": false, <1>
   "items": [
      {  "delete": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 2,
            "status":   200,
            "found":    true
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 3,
            "status":   201
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "EiwfApScQiiy7TIKFxRCTw",
            "_version": 1,
            "status":   201
      }},
      {  "update": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 4,
            "status":   200
      }}
   ]
}}
```

- <1> 所有子请求都成功完成。

每个子请求都被独立的执行，所以一个子请求的错误并不影响其它请求。如果任何一个请求失败，顶层的`error`标记将被设置为`true`，然后错误的细节将在相应的请求中被报告：

```Javascript
POST /_bulk
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "Cannot create - it already exists" }
{ "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "But we can update it" }
```

响应中我们将看到`create`文档`123`失败了，因为文档已经存在，但是后来的在`123`上执行的`index`请求成功了：

```Javascript
{
   "took": 3,
   "errors": true, <1>
   "items": [
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "status":   409, <2>
            "error":    "DocumentAlreadyExistsException <3>
                        [[website][4] [blog][123]:
                        document already exists]"
      }},
      {  "index": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 5,
            "status":   200 <4>
      }}
   ]
}
```

- <1> 一个或多个请求失败。
- <2> 这个请求的HTTP状态码被报告为`409 CONFLICT`。
- <3> 错误消息说明了什么请求错误。
- <4> 第二个请求成功了，状态码是`200 OK`。

这些说明`bulk`请求不是原子操作——它们不能实现事务。每个请求操作时分开的，所以每个请求的成功与否不干扰其它操作。

## 不要重复

你可能在同一个`index`下的同一个`type`里批量索引日志数据。为每个文档指定相同的元数据是多余的。就像`mget` API，`bulk`请求也可以在URL中使用`/_index`或`/_index/_type`:

```Javascript
POST /website/_bulk
{ "index": { "_type": "log" }}
{ "event": "User logged in" }
```

你依旧可以覆盖元数据行的`_index`和`_type`，在没有覆盖时它会使用URL中的值作为默认值：

```Javascript
POST /website/log/_bulk
{ "index": {}}
{ "event": "User logged in" }
{ "index": { "_type": "blog" }}
{ "title": "Overriding the default type" }
```

## 多大才算太大？

整个批量请求需要被加载到接受我们请求节点的内存里，所以请求越大，给其它请求可用的内存就越小。有一个最佳的`bulk`请求大小。超过这个大小，性能不再提升而且可能降低。

最佳大小，当然并不是一个固定的数字。它完全取决于你的硬件、你文档的大小和复杂度以及索引和搜索的负载。幸运的是，这个**最佳点(sweetspot)**还是容易找到的：

试着批量索引标准的文档，随着大小的增长，当性能开始降低，说明你每个批次的大小太大了。开始的数量可以在1000~5000个文档之间，如果你的文档非常大，可以使用较小的批次。

通常着眼于你请求批次的物理大小是非常有用的。一千个1kB的文档和一千个1MB的文档大不相同。一个好的批次最好保持在5-15MB大小间。

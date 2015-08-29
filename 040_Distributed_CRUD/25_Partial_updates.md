## 局部更新文档

`update` API 结合了之前提到的读和写的模式。

![局部更新文档](https://raw.githubusercontent.com/looly/elasticsearch-definitive-guide-cn/master/images/elas_0404.png)

下面我们罗列执行局部更新必要的顺序步骤：

1. 客户端给`Node 1`发送更新请求。
2. 它转发请求到主分片所在节点`Node 3`。
3. `Node 3`从主分片检索出文档，修改`_source`字段的JSON，然后在主分片上重建索引。如果有其他进程修改了文档，它以`retry_on_conflict`设置的次数重复步骤3，都未成功则放弃。
4. 如果`Node 3`成功更新文档，它同时转发文档的新版本到`Node 1`和`Node 2`上的复制节点以重建索引。当所有复制节点报告成功，`Node 3`返回成功给请求节点，然后返回给客户端。

`update` API还接受《新建、索引和删除》章节提到的`routing`、`replication`、`consistency`和`timout`参数。

> ### 基于文档的复制

> 当主分片转发更改给复制分片时，并不是转发更新请求，而是转发整个文档的新版本。记住这些修改转发到复制节点是异步的，它们并不能保证到达的顺序与发送相同。如果Elasticsearch转发的仅仅是修改请求，修改的顺序可能是错误的，那得到的就是个损坏的文档。

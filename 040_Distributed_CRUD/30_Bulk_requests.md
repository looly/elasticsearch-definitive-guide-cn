## 多文档模式

`mget`和`bulk` API与单独的文档类似。差别是请求节点知道每个文档所在的分片。它把多文档请求拆成**每个分片**的对文档请求，然后转发每个参与的节点。

一旦接收到每个节点的应答，然后整理这些响应组合为一个单独的响应，最后返回给客户端。

![通过mget检索多个文档](https://raw.githubusercontent.com/looly/elasticsearch-definitive-guide-cn/master/images/elas_0405.png)

下面我们将罗列通过一个`mget`请求检索多个文档的顺序步骤：

1. 客户端向`Node 1`发送`mget`请求。
2. `Node 1`为每个分片构建一个多条数据检索请求，然后转发到这些请求所需的主分片或复制分片上。当所有回复被接收，`Node 1`构建响应并返回给客户端。

`routing` 参数可以被`docs`中的每个文档设置。

![通过打包批量修改文档](https://raw.githubusercontent.com/looly/elasticsearch-definitive-guide-cn/master/images/elas_0406.png)

下面我们将罗列使用一个`bulk`执行多个`create`、`index`、`delete`和`update`请求的顺序步骤：

1. 客户端向`Node 1`发送`bulk`请求。
2. `Node 1`为每个分片构建批量请求，然后转发到这些请求所需的主分片上。
3. 主分片一个接一个的按序执行操作。当一个操作执行完，主分片转发新文档（或者删除部分）给对应的复制节点，然后执行下一个操作。一旦所有复制节点报告所有操作已成功完成，节点就报告success给请求节点，后者(请求节点)整理响应并返回给客户端。

`bulk` API还可以在最上层使用`replication`和`consistency`参数，`routing`参数则在每个请求的元数据中使用。



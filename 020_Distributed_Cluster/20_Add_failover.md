## 增加故障转移

在单一节点上运行意味着有单点故障的风险——没有数据冗余备份。幸运的是我们可以启动另一个节点来保护我们的数据不被丢失。

> ### 启动第二个节点

> 为了测试在增加第二个节点后发生了什么，你可以使用与第一个节点相同的方式启动第二个节点（《运行Elasticsearch》一章），而且在同一个目录——多个节点可以分享同一个目录。

> 只要第二个节点与第一个节点有相同的`cluster.name`（请看`./config/elasticsearch.yml`文件），它就能自动发现并加入第一个节点的集群。如果没有，检查日志找出哪里出了问题。这可能是网络广播被禁用，或者防火墙阻止了节点通信。

如果我们启动了第二个节点，这个集群应该叫做**双节点集群(cluster-two-nodes)**

双节点集群——所有的主分片和复制分片都被分配:
![双节点集群](../images/02-03_two_nodes.png)

第二个节点加入集群时，三个**复制碎片(replica shards)**已经被分配了——与三个主分片一一对应。那意味着在丢失一个节点的情况下依旧可以保证数据的完整性。

一些新的被索引的文档将首先被存储在主分片中，然后平行复制到关联的复制节点上。这可以确保我们的数据在主节点和复制节点上都可以被检索。

`cluster-health`现在的状态是`green`，这意味着所有的6个分片（三个主分片和三个复制分片）都已可用：

```Javascript
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       2,
   "number_of_data_nodes":  2,
   "active_primary_shards": 3,
   "active_shards":         6,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```

- <1> 集群的`status`是`green`.

我们的集群不仅是全功能的，而且是**高可用**的。

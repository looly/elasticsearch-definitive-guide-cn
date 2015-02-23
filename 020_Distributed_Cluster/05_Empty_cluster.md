## 空集群

如果我们启动一个单独的节点，没有数据和索引，这个集群我们称作“只有一个空节点的集群”。

![A cluster with one empty node](https://raw.githubusercontent.com/looly/elasticsearch-definitive-guide-cn/master/images/02-01_cluster.png)

一个**节点(node)**就是一个Elasticsearch实例，而一个**集群(cluster)**由一个或多个节点组成，它们具有相同的`cluster.name`，它们协同工作，分享数据和负载。当有新的节点加入或者删除节点，集群就会感知到并平衡数据。

集群中一个节点会被选举为**主节点(master)**,它用来管理集群中的一些变更，例如新建或删除索引、增加或移除节点等。主节点不涉及文档级别的更改或搜索，这意味着一个主节点随着流量的增长而成为集群的瓶颈。任何节点可以成为主节点。我们例子中的集群只有一个节点，所以它会充当主节点的角色。

做为用户，我们能够与**集群中的任何节点(any node in the cluster)**通信，包括主节点。任何一个节点互相知道文档存在于哪个节点上，它们可以转发请求到我们需要数据所在的节点上。我们通信的节点负责收集各节点返回的数据，最后一起返回给客户端。这一切都由Elasticsearch透明的管理。


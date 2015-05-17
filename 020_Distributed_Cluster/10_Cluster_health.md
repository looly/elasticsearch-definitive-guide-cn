## 集群健康

在Elasticsearch集群中可以监控统计很多信息，但是只有一个是最重要的：**集群健康(cluster health)**。集群健康有三种状态：`green`、`yellow`或`red`。

```Javascript
GET /_cluster/health
```
在一个没有索引的空集群中运行如上查询，将返回这些信息：

```Javascript
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```
- <1> `status` 是我们最感兴趣的字段

`status`字段提供一个综合的指标来表示集群的的服务状况。三种颜色各自的含义：

| 颜色     | 意义                                       |
| -------- | ----------------------------------------  |
| `green`  | 所有主要分片和复制分片都可用 |
| `yellow` | 所有主要分片可用，但不是所有复制分片都可用 |
| `red`    | 不是所有的主要分片都可用                   |

在接下来的章节，我们将说明什么是**主要分片(primary shard)**和**复制分片(replica shard)**，并说明这些颜色（状态）在实际环境中的意义。

##查询阶段
在初始化_查询阶段_（_query phase_），查询被向索引中的每个分片副本（原本或副本）广播。每个分片在本地执行搜索并且建立了匹配document的_优先队列_（_priority queue_）。
> ####优先队列

> 一个_优先队列_（_priority queue_ is）只是一个存有_前n个_（_top-n_）匹配document的有序列表。这个优先队列的大小由分页参数from和size决定。例如，下面这个例子中的搜索请求要求优先队列要能够容纳100个document

``` JavaScript
GET /_search
{
    "from": 90,
    "size": 10
}
```

这个查询的过程被描述在图分布式搜索查询阶段中。

![Query phase of distributed search](../images/elas_0901.png)

图1 分布式搜索查询阶段

查询阶段包含以下三步：

1.客户端发送一个`search（搜索）`请求给`Node 3`,`Node 3`创建了一个长度为`from+size`的空优先级队列。
2.`Node 3` 转发这个搜索请求到索引中每个分片的原本或副本。每个分片在本地执行这个查询并且结果将结果到一个大小为`from+size`的有序本地优先队列里去。
3.每个分片返回document的ID和它优先队列里的所有document的排序值给协调节点`Node 3`。`Node 3`把这些值合并到自己的优先队列里产生全局排序结果。

当一个搜索请求被发送到一个节点Node，这个节点就变成了协调节点。这个节点的工作是向所有相关的分片广播搜索请求并且把它们的响应整合成一个全局的有序结果集。这个结果集会被返回给客户端。

第一步是向索引里的每个节点的分片副本广播请求。就像document的`GET`请求一样，搜索请求可以被每个分片的原本或任意副本处理。这就是更多的副本（当结合更多的硬件时）如何提高搜索的吞吐量的方法。对于后续请求，协调节点会轮询所有的分片副本以分摊负载。

每一个分片在本地执行查询和建立一个长度为`from+size`的有序优先队列——这个长度意味着它自己的结果数量就足够满足全局的请求要求。分片返回一个轻量级的结果列表给协调节点。只包含documentID值和排序需要用到的值，例如`_score`。

协调节点将这些分片级的结果合并到自己的有序优先队列里。这个就代表了最终的全局有序结果集。到这里，查询阶段结束。

整个过程类似于归并排序算法，先分组排序再归并到一起，对于这种分布式场景非常适用。
> ###注意

> 一个索引可以由一个或多个原始分片组成，所以一个对于单个索引的搜索请求也需要能够把来自多个分片的结果组合起来。一个对于
_多（multiple）_或_全部（all）_索引的搜索的工作机制和这完全一致——仅仅是多了一些分片而已。


<!--
=== Query Phase

During the initial _query phase_,  the((("distributed search execution", "query phase")))((("query phase of distributed search"))) query is broadcast to a shard copy (a
primary or replica shard) of every shard in the index. Each shard executes
the search locally and ((("priority queue")))builds a _priority queue_ of matching documents.

.Priority Queue
****

A _priority queue_ is just a sorted list that holds the _top-n_ matching
documents. The size of the priority queue depends on the pagination
parameters `from` and `size`.  For example, the following search request
would require a priority queue big enough to hold 100 documents:

[source,js]
--------------------------------------------------
GET /_search
{
    "from": 90,
    "size": 10
}
--------------------------------------------------
****

The query phase process is depicted in <<img-distrib-search>>.

[[img-distrib-search]]
.Query phase of distributed search
image::images/elas_0901.png["Query phase of distributed search"]

The query phase consists of the following three steps:

1. The client sends a `search` request to `Node 3`, which creates an empty
   priority queue of size `from + size`.

2. `Node 3` forwards the search request to a primary or replica copy of every
   shard in the index. Each shard executes the query locally and adds the
   results into a local sorted priority queue of size `from + size`.

3. Each shard returns the doc IDs and sort values of all the docs in its
   priority queue to the coordinating node, `Node 3`, which merges these
   values into its own priority queue to produce a globally sorted list of
   results.

When a search request is sent to a node, that node becomes the coordinating
node.((("nodes", "coordinating node for search requests"))) It is the job of this node to broadcast the search request to all
involved shards, and to gather their responses into a globally sorted result
set that it can return to the client.

The first step is to broadcast the request to a shard copy of every node in
the index. Just like <<distrib-read,document `GET` requests>>, search requests
can be handled by a primary shard or by any of its replicas.((("shards", "handling search requests"))) This is how more
replicas (when combined with more hardware) can increase search throughput.
A coordinating node will round-robin through all shard copies on subsequent
requests in order to spread the load.

Each shard executes the query locally and builds a sorted priority queue of
length `from + size`&#x2014;in other words, enough results to satisfy the global
search request all by itself. It returns a lightweight list of results to the
coordinating node, which contains just the doc IDs and any values required for
sorting, such as the `_score`.

The coordinating node merges these shard-level results into its own sorted
priority queue, which represents the globally sorted result set. Here the query
phase ends.

[NOTE]
====
An index can consist of one or more primary shards,((("indices", "multi-index search"))) so a search request
against a single index needs to be able to combine the results from multiple
shards. A search against _multiple_ or _all_ indices works in exactly the same
way--there are just more shards involved.
====
-->

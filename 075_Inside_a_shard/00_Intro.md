[[inside-a-shard]]
== Inside a Shard

In <<distributed-cluster>>, we introduced the _shard_, and described((("shards"))) it as a
low-level _worker unit_. But what exactly _is_ a shard and how does it work?
In this chapter, we answer these questions:

* Why is search _near_ real-time?
* Why are document CRUD (create-read-update-delete) operations _real-time_?
* How does Elasticsearch ensure that the changes you make are durable, that
  they won't be lost if there is a power failure?
* Why does deleting documents not free up space immediately?
* What do the `refresh`, `flush`, and `optimize` APIs do, and when should
  you use them?

The easiest way to understand how a shard functions today is to start with a
history lesson. We will look at the problems that needed to be solved in order
to provide a distributed durable data store with near real-time search and
analytics.

.Content Warning
****

The information presented in this chapter is for your interest. You are not required to
understand and remember all the detail in order to use Elasticsearch. Read
this chapter to gain a taste for how things work, and to know where the
information is in case you need to refer to it in the future, but don't be
overwhelmed by the detail.

****

# 入门
在[分布式集群](020_Distributed_Cluster/00_Intro.md)中，我们介绍了分片，把它描述为底层的工作单元。但分片到底是什么，它怎样工作？在这章节，我们将回答这些问题：

  *为什么搜索是近实时的？
  *为什么文档的CRUD操作是实时的？
ES怎样保证更新持久化，即使断电也不会丢失？
为什么删除文档不会立即释放空间？
什么是refresh，flush，优化API，以及什么时候你该使用它们？

#分布式搜索的执行方式
在继续之前，我们将绕道讲一下搜索是如何在分布式环境中执行的。 <!--"distributed search execution"--> 它比我们之前讲的基础的_增删改查_(_create-read-update-delete_ ，CRUD)<!--"CRUD (create-read-update-delete) operations"-->请求要复杂一些。

> ####注意：

> 本章的信息只是出于兴趣阅读，使用Elasticsearch并不需要理解和记住这里的所有细节。

> 阅读这一章只是增加对系统如何工作的了解，并让你知道这些信息以备以后参考，所以别淹没在细节里。

一个CRUD操作只处理一个单独的文档。文档的唯一性由`_index`, `_type`和`routing-value`（通常默认是该文档的`_id`）的组合来确定。这意味着我们可以准确知道集群中的哪个分片持有这个文档。

由于不知道哪个文档会匹配查询（文档可能存放在集群中的任意分片上），所以搜索需要一个更复杂的模型。一个搜索不得不通过查询每一个我们感兴趣的索引的分片副本，来看是否含有任何匹配的文档。

但是，找到所有匹配的文档只完成了这件事的一半。在搜索（`search`）API返回一页结果前，来自多个分片的结果必须被组合放到一个有序列表中。因此，搜索的执行过程分两个阶段，称为_查询然后取回_（_query then fetch_）。

<!--

[[distributed-search]]
== Distributed Search Execution

Before moving on, we are going to take a detour and talk about how search is
executed in a distributed environment.((("distributed search execution")))  It is a bit more complicated than the
basic _create-read-update-delete_ (CRUD) requests((("CRUD (create-read-update-delete) operations"))) that we discussed in
<<distributed-docs>>.

.Content Warning
****

The information presented in this chapter is for your interest. You are not required to
understand and remember all the detail in order to use Elasticsearch.

Read this chapter to gain a taste for how things work, and to know where the
information is in case you need to refer to it in the future, but don't be
overwhelmed by the detail.

****

A CRUD operation deals with a single document that has a unique combination of
`_index`, `_type`, and <<routing-value,`routing` values>> (which defaults to the
document's `_id`). This means that we know exactly which shard in the cluster
holds that document.

Search requires a more complicated execution model because we don't know which
documents will match the query: they could be on any shard in the cluster. A
search request has to consult a copy of every shard in the index or indices
we're interested in to see if they have any matching documents.

But finding all matching documents is only half the story. Results from
multiple shards must be combined into a single sorted list before the `search`
API can return a ``page'' of results. For this reason, search is executed in a
two-phase process called _query then fetch_.

-->

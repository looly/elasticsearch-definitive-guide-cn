##取回阶段

查询阶段辨别出那些满足搜索请求的document，但我们仍然需要取回那些document本身。这就是取回阶段的工作，如图分布式搜索的取回阶段所示。

![Fetch phase of distributed search](../images/elas_0902.png)

图2 分布式搜索取回阶段

分发阶段由以下步骤构成：

1.协调节点辨别出哪个document需要取回，并且向相关分片发出`GET`请求。

2.每个分片加载document并且根据需要_丰富（enrich）_它们，然后再将document返回协调节点。

3.一旦所有的document都被取回，协调节点会将结果返回给客户端。

协调节点先决定哪些document是_实际（actually）_需要取回的。例如，我们指定查询```{ "from": 90, "size": 10 }```，那么前90条将会被丢弃，只有之后的10条会需要取回。这些document可能来自与原始查询请求相关的某个、某些或者全部分片。

协调节点为每个持有相关document的分片建立多点get请求然后发送请求到处理查询阶段的分片副本。

分片加载document主体——`_source` field。如果需要，还会根据元数据丰富结果和高亮搜索片断。一旦协调节点收到所有结果，会将它们汇集到单一的回答响应里，这个响应将会返回给客户端。
###深分页
****
查询然后取回过程虽然支持通过使用`from`和`size`参数进行分页，但是_要在有限范围内（within limited）_。还记得每个分片必须构造一个长度为`from+size`的优先队列吧，所有这些都要传回协调节点。这意味着协调节点要通过对`分片数量 * (from + size)`个document进行排序来找到正确的`size`个document。

根据document的数量，分片的数量以及所使用的硬件，对10,000到50,000条结果（1,000到5,000页）深分页是可行的。但是对于足够大的`from`值，排序过程将会变得非常繁重，会使用巨大量的CPU，内存和带宽。因此，强烈不建议使用深分页。

在实际中，“深分页者”也是很少的一部人。一般人会在翻了两三页后就停止翻页，并会更改搜索标准。那些不正常情况通常是机器人或者网络爬虫的行为。它们会持续不断地一页接着一页地获取页面直到服务器到崩溃的边缘。

如果你确实需要从集群里获取大量documents，你可以通过设置搜索类型`scan`禁用排序，来高效地做这件事。这一点将在后面的章节讨论。

****







<!--
=== Fetch Phase

The query phase identifies which documents satisfy((("distributed search execution", "fetch phase")))((("fetch phase of distributed search"))) the search request, but we
still need to retrieve the documents themselves. This is the job of the fetch
phase, shown in <<img-distrib-fetch>>.

[[img-distrib-fetch]]
.Fetch phase of distributed search
image::images/elas_0902.png["Fetch Phase of distributed search"]

The distributed phase consists of the following steps:

1. The coordinating node identifies which documents need to be fetched and
   issues a multi `GET` request to the relevant shards.

2. Each shard loads the documents and _enriches_ them, if required, and then
   returns the documents to the coordinating node.

3. Once all documents have been fetched, the coordinating node returns the
   results to the client.

The coordinating node first decides which documents _actually_ need to be
fetched. For instance, if our query specified `{ "from": 90, "size": 10 }`,
the first 90 results would be discarded and only the next 10 results would
need to be retrieved. These documents may come from one, some, or all of the
shards involved in the original search request.

The coordinating node builds a <<distrib-multi-doc,multi-get request>> for
each shard that holds a pertinent document and sends the request to the same
shard copy that handled the query phase.

The shard loads the document bodies--the `_source` field--and, if
requested, enriches the results with metadata and
<<highlighting-intro,search snippet highlighting>>.
Once the coordinating node receives all results, it assembles them into a
single response that it returns to the client.

.Deep Pagination
****

The query-then-fetch process supports pagination with the `from` and `size`
parameters, but _within limits_. ((("size parameter")))((("from parameter")))((("pagination", "supported by query-then-fetch process")))((("deep paging, problems with"))) Remember that each shard must build a priority
queue of length `from + size`, all of which need to be passed back to
the coordinating node. And the coordinating node needs to sort through
`number_of_shards * (from + size)` documents in order to find the correct
`size` documents.

Depending on the size of your documents, the number of shards, and the
hardware you are using, paging 10,000 to 50,000 results (1,000 to 5,000 pages)
deep should be perfectly doable. But with big-enough `from` values, the
sorting process can become very heavy indeed, using vast amounts of CPU,
memory, and bandwidth.  For this reason, we strongly advise against deep paging.

In practice, ``deep pagers'' are seldom human anyway.  A human will stop
paging after two  or three pages and will change the search criteria. The
culprits are usually bots or web spiders that tirelessly keep fetching page
after page until your servers crumble at the knees.

If you _do_ need to fetch large numbers of docs from your cluster, you can
do so efficiently by disabling sorting with the `scan` search type,
which we discuss <<scan-scroll,later in this chapter>>.

****

-->

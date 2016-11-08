<!--秀川译-->
### 关联失效

在我们去讨论多字段检索中的更复杂的查询前，让我们顺便先解释一下为什么我们只用一个主分片来创建索引。

有时有的新手会开一个问题说通过相关性排序没有效果，并且提供了一小段复制的结果：该用户创建了一些文档，执行了一个简单的查询，结果发现相关性较低的结果排在了相关性较高的结果的前面。

为了理解为什么会出现这样的结果，我们假设用两个分片创建一个索引，以及索引10个文档，6个文档包含词 `foo`，这样可能会出现分片1中有3个文档包含 `foo`，分片2中也有三个文档包含 `foo`。换句话说，我们的文档做了比较好的分布式。

在相关性介绍这一节，我们描述了Elasticsearch默认的相似算法，叫做词频率/反转文档频率（TF/IDF）。词频率是一个词在我们当前查询的文档的字段中出现的次数。出现的次数越多，相关性就越大。反转文档频率指的是该索引中所有文档数与出现这个词的文件数的百分比，词出现的频率越大，IDF越小。

然而，由于性能问题，Elasticsearch不通过索引中所有的文档计算IDF。每个分片会为分片中所有的文档计算一个本地的IDF取而代之。

因为我们的文档做了很好的分布式，每个分片的IDF是相同的。现在假设5个包含`foo`的文档在分片1中，以及其他6各文档在分片2中。在这个场景下，词`foo`在第一个分片中是非常普通的（因此重要性较小），但是在另一个分片中是很稀少的（因此重要性较高）。这些区别在IDF中就会产生不正确的结果。

事实证明，这并不是一个问题。你索引越多的文档，本地IDF和全局IDF的区别就会越少。在实际工作的数据量下，本地IDF立刻能够很好的工作（With real-world
volumes of data, the local IDFs soon even out，不知道这么翻译合不合适）。所以问题不是因为关联失效，而是因为数据太少。

为了测试的目的，对于这个问题，有两种方法可以奏效。第一种方法是创建一个只有一个主分片的索引，像我们介绍`match`查询那节一样做。如果你只有一个分片，那么本地IDF就是全局IDF。

第二种方法是在你们请求中添加`?search_type=dfs_query_then_fetch`。`dfs`就是*Distributed Frequency Search*，并且它会告诉Elasticsearch检查每一个分片的本地IDF为了计算整个索引的全局IDF。

> 提示：不要把`dfs_query_then_fetch`用于生产环境。它实在是没有必要。只要有足够的数据就能够确保词频率很好的分布。没有理由在每个你要执行的查询中添加额外的DFS步骤。


<!--
[[relevance-is-broken]]
=== Relevance Is Broken!

Before we move on to discussing more-complex queries in
<<multi-field-search>>, let's make a quick detour to explain why we
<<match-test-data,created our test index>> with just one primary shard.

Every now and again a new user opens an issue claiming that sorting by
relevance((("relevance", "differences in IDF producing incorrect results"))) is broken and offering a short reproduction: the user indexes a few
documents, runs a simple query, and finds apparently less-relevant results
appearing above more-relevant results.

To understand why this happens, let's imagine that we create an index with two
primary shards and we index ten documents, six of which contain the word `foo`.
It may happen that shard 1 contains three of the `foo` documents and shard
2 contains the other three.  In other words, our documents are well distributed.

In <<relevance-intro>>, we described the default similarity algorithm used in
Elasticsearch, ((("Term Frequency/Inverse Document Frequency  (TF/IDF) similarity algorithm")))called _term frequency / inverse document frequency_ or TF/IDF.
Term frequency counts the number of times a term appears within the field we are
querying in the current document.  The more times it appears, the more
relevant is this document. The _inverse document frequency_ takes((("inverse document frequency")))((("IDF", see="inverse document frequency"))) into account
how often a term appears as a percentage of _all the documents in the index_.
The more frequently the term appears, the less weight it has.

However, for performance reasons, Elasticsearch doesn't calculate the IDF
across all documents in the index.((("shards", "local inverse document frequency (IDF)"))) Instead, each shard calculates a local IDF
for the documents contained _in that shard_.

Because our documents are well distributed, the IDF for both shards will be
the same.  Now imagine instead that five of the `foo` documents are on shard 1,
and the sixth document is on shard 2.  In this scenario, the term `foo` is
very common on one shard (and so of little importance), but rare on the other
shard (and so much more important). These differences in IDF can produce
incorrect results.

In practice, this is not a problem. The differences between local and  global
IDF diminish the more documents that you add to the index. With real-world
volumes of data, the local IDFs soon even out. The problem is not that
relevance is broken but that there is too little data.

For testing purposes, there are two ways we can work around this issue. The
first is to create an index with one primary shard, as we did in the section
introducing the <<match-query,`match` query>>. If you have only one shard, then
the local IDF _is_ the global IDF.

The second workaround is to add `?search_type=dfs_query_then_fetch` to your
search requests. The `dfs` stands((("search_type", "dfs_query_then_fetch")))((("dfs_query_then_fetch search type")))((("DFS (Distributed Frequency Search)"))) for _Distributed Frequency Search_, and it
tells Elasticsearch to first retrieve the local IDF from each shard in order
to calculate the global IDF across the whole index.

TIP: Don't use `dfs_query_then_fetch` in production.  It really isn't
required. Just having enough data will ensure that your term frequencies are
well distributed. There is no reason to add this extra DFS step to every query
that you run.

-->

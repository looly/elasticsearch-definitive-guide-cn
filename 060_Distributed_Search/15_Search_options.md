##搜索选项

一些查询字符串（query-string）可选参数能够影响搜索过程。

####preference（偏爱）

`preference`参数允许你控制使用哪个分片或节点来处理搜索请求。她接受如下一些参数 `_primary`， `_primary_first`， `_local`， `_only_node:xyz`， `_prefer_node:xyz`和`_shards:2,3`。这些参数在文档[搜索偏好（search preference）](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-request-preference.html)里有详细描述。

然而通常最有用的值是一些随机字符串，它们可以避免结果震荡问题（the _bouncing results_ problem）。

#####结果震荡（Bouncing Results）

* 想像一下，你正在按照`timestamp`字段来对你的结果排序，并且有两个document有相同的timestamp。由于搜索请求是在所有有效的分片副本间轮询的，这两个document可能在原始分片里是一种顺序，在副本分片里是另一种顺序。

* 这就是被称为_结果震荡（bouncing  results）_的问题：用户每次刷新页面，结果顺序会发生变化。避免这个问题方法是对于同一个用户总是使用同一个分片。方法就是使用一个随机字符串例如用户的会话ID（session ID）来设置`preference`参数。

###timeout（超时）
通常，协调节点会等待接收所有分片的回答。如果有一个节点遇到问题，它会拖慢整个搜索请求。

`timeout`参数告诉协调节点最多等待多久，就可以放弃等待而将已有结果返回。返回部分结果总比什么都没有好。


搜索请求的返回将会指出这个搜索是否超时，以及有多少分片成功答复了：

``` js
    ...
    "timed_out":     true,  (1)
    "_shards": {
       "total":      5,
       "successful": 4,
       "failed":     1     (2)
    },
    ...
```
--------------------------------------------------
(1) 搜索请求超时。

(2) 五个分片中有一个没在超时时间内答复。

如果一个分片的所有副本都因为其他原因失败了——也许是因为硬件故障——这个也同样会反映在该答复的`_shards`部分里。

### routing（路由选择）
在路由值那节里，我们解释了如何在建立索引时提供一个自定义的`routing`参数来保证所有相关的document（如属于单个用户的document）被存放在一个单独的分片中。在搜索时，你可以指定一个或多个`routing` 值来限制只搜索那些分片而不是搜索index里的全部分片：

``` js
GET /_search?routing=user_1,user2
```
这个技术在设计非常大的搜索系统时就会派上用场了。我们在规模（scale）那一章里详细讨论它。

### search_type（搜索类型）
虽然`query_then_fetch`是默认的搜索类型，但也可以根据特定目的指定其它的搜索类型，例如：

``` js
GET /_search?search_type=count
```

___count（计数）___

`count（计数）`搜索类型只有一个`query（查询）`的阶段。当不需要搜索结果只需要知道满足查询的document的数量时，可以使用这个查询类型。


___query_and_fetch（查询并且取回）___

`query_and_fetch（查询并且取回）`搜索类型将查询和取回阶段合并成一个步骤。这是一个内部优化选项，当搜索请求的目标只是一个分片时可以使用，例如指定了`routing（路由选择）`值时。虽然你可以手动选择使用这个搜索类型，但是这么做基本上不会有什么效果。


___dfs_query_then_fetch___ 和 ___dfs_query_and_fetch___

`dfs`搜索类型有一个预查询的阶段，它会从全部相关的分片里取回项目频数来计算全局的项目频数。我们将在relevance-is-broken（相关性被破坏）里进一步讨论这个。


___scan（扫描）___

`scan（扫描）`搜索类型是和`scroll（滚屏）`API连在一起使用的，可以高效地取回巨大数量的结果。它是通过禁用排序来实现的。我们将在下一节_scan-and-scroll（扫描和滚屏）_里讨论它。



<!--
=== Search Options

A few ((("search options")))optional query-string parameters can influence the search process.

==== preference

The `preference` parameter allows((("preference parameter")))((("search options", "preference"))) you to control which shards or nodes are
used to handle the search request. It accepts values such as `_primary`,
`_primary_first`, `_local`, `_only_node:xyz`, `_prefer_node:xyz`, and
`_shards:2,3`, which are explained in detail on the
http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-request-preference.html[search `preference`]
documentation page.

However, the most generally useful value is some arbitrary string, to avoid
the _bouncing results_ problem.((("bouncing results problem")))

[[bouncing-results]]
.Bouncing Results
****

Imagine that you are sorting your results by a `timestamp` field, and
two documents have the same timestamp.  Because search requests are
round-robined between all available shard copies, these two documents may be
returned in one order when the request is served by the primary, and in
another order when served by the replica.

This is known as the _bouncing results_ problem: every time the user refreshes
the page, the results appear in a different order. The problem can be avoided by always using the same shards for the same user,
which can be done by setting the `preference` parameter to an arbitrary string
like the user's session ID.

****

==== timeout

By default, the coordinating node waits((("search options", "timeout"))) to receive a response from all shards.
If one node is having trouble, it could slow down the response to all search
requests.

The `timeout` parameter tells((("timeout parameter"))) the coordinating node how long it should wait
before giving up and just returning the results that it already has. It can be
better to return some results than none at all.

The response to a search request will indicate whether the search timed out and
how many shards responded successfully:

[source,js]
--------------------------------------------------
    ...
    "timed_out":     true,  <1>
    "_shards": {
       "total":      5,
       "successful": 4,
       "failed":     1 <2>
    },
    ...
--------------------------------------------------
<1> The search request timed out.
<2> One shard out of five failed to respond in time.

If all copies of a shard fail for other reasons--perhaps because of a
hardware failure--this will also be reflected in the `_shards` section of
the response.

[[search-routing]]
==== routing

In <<routing-value>>, we explained how a custom `routing` parameter((("search options", "routing")))((("routing parameter"))) could be
provided at index time to ensure that all related documents, such as the
documents belonging to a single user, are stored on a single shard.  At search
time, instead of searching on all the shards of an index, you can specify
one or more `routing` values to limit the search to just those shards:

[source,js]
--------------------------------------------------
GET /_search?routing=user_1,user2
--------------------------------------------------

This technique comes in handy when designing very large search systems, and we
discuss it in detail in <<scale>>.

[[search-type]]
==== search_type

While `query_then_fetch` is the default((("query_then_fetch search type")))((("search options", "search_type")))((("search_type"))) search type, other search types can
be specified for particular purposes, for example:

[source,js]
--------------------------------------------------
GET /_search?search_type=count
--------------------------------------------------

`count`::

The `count` search type has only a `query` phase.((("count search type")))  It can be used when you
don't need search results, just a document count or
<<aggregations,aggregations>> on documents matching the query.

`query_and_fetch`::

The `query_and_fetch` search type ((("query_and_fetch serch type")))combines the query and fetch phases into a
single step.  This is an internal optimization that is used when a search
request targets a single shard only, such as when a
<<search-routing,`routing`>> value has been specified. While you can choose
to use this search type manually, it is almost never useful to do so.

`dfs_query_then_fetch` and `dfs_query_and_fetch`::

The `dfs` search types((("dfs search types"))) have a prequery phase that fetches the term
frequencies from all involved shards in order to calculate global term
frequencies. We discuss this further in <<relevance-is-broken>>.

`scan`::

The `scan` search type is((("scan search type"))) used in conjunction with the `scroll` API ((("scroll API")))to
retrieve large numbers of results efficiently. It does this by disabling
sorting.  We discuss _scan-and-scroll_ in the next section.





-->

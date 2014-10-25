#空搜索

最近本的搜索API表单是**空搜索(empty search)**，它没有指定任何的查询条件，只返回集群索引中的所有文档：
```Javascript
GET /_search
```

The response (edited for brevity) looks something like this:
响应内容（为了编辑简洁）类似于这样：

```Javascript
{
   "hits" : {
      "total" :       14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       },
        ... 9 RESULTS REMOVED ...
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}
```

## `hits`

响应中最重要的部分是`hits`，它包含了`total`字段来表示匹配到的文档总数，`hits`数组还包含了匹配到的前10条数据。

Each result in the `hits` array contains the `_index`, `_type` and `_id` of
the document, plus the `_source` field.  This means that the whole document is
immediately available to us directly from the search results. This is unlike
other search engines which return just the document ID, requiring you to fetch
the document itself in a separate step.

Each element also has a `_score`.  This is the _relevance score_, which is a
measure of how well the document matches the query.  By default, results are
returned with the most relevant documents first; that is, in descending order
of `_score`. In this case, we didn't specify any query so all documents are
equally relevant, hence the neutral `_score` of `1` for all results.

The `max_score` value is the highest `_score` of any document that matches our
query.

==== `took`

The `took` value tells us how many milliseconds the entire search request took
to execute.

==== `shards`

The `_shards` element tells us the `total` number of shards that were involved
in the query and, of them, how many were `successful` and how many `failed`.
We wouldn't normally expect shards to fail, but it can happen. If we were to
suffer a major disaster in which we lost both the primary and the replica copy
of the same shard, there would be no copies of that shard available to respond
to search requests. In this case, Elasticsearch would report the shard as
`failed`, but continue to return results from the remaining shards.

==== `timeout`

The `timed_out` value tells us whether the query timed out or not.  By
default, search requests do not timeout.  If low response times are more
important to you than complete results, you can specify a `timeout` as `10`
or `"10ms"` (10 milliseconds), or `"1s"` (1 second):

[source,js]
--------------------------------------------------
GET /_search?timeout=10ms
--------------------------------------------------


Elasticsearch will return any results that it has managed to gather from
shards which responded before the request timed out.

.Timeout is not a circuit breaker
[WARNING]
================================================

It should be noted that this `timeout` does not halt the execution of the
query, it merely tells the coordinating node to return the results collected
_so far_ and to close the connection.  In the background, other shards may
still be processing the query even though results have been sent.

Use the timeout because it is important to your SLA, not because you want
to abort the execution of long running queries.

================================================


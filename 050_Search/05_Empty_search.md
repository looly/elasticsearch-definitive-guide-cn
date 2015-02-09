#空搜索

最基本的搜索API表单是**空搜索(empty search)**，它没有指定任何的查询条件，只返回集群索引中的所有文档：
```Javascript
GET /_search
```

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

`hits`数组中的每个结果都包含`_index`、`_type`和文档的`_id`字段，被加入到`_source`字段中这意味着在搜索结果中我们将可以直接使用全部文档。这不像其他搜索引擎只返回文档ID，需要你单独去获取文档。

每个节点都有一个`_score`字段，这是**相关性得分(relevance score)**，它衡量了文档与查询的匹配程度。默认的，返回的结果中关联性最大的文档排在首位；这意味着，它是按照`_score`降序排列的。这种情况下，我们没有指定任何查询，所以所有文档的相关性是一样的，因此所有结果的`_score`都是取得一个中间值`1`

`max_score`指的是所有文档匹配查询中`_score`的最大值。

## `took`

`took`告诉我们整个搜索请求花费的毫秒数。

## `shards`

`_shards`节点告诉我们参与查询的分片数（`total`字段），有多少是成功的（`successful`字段），有多少的是失败的（`failed`字段）。通常我们不希望分片失败，不过这个有可能发生。如果我们遭受一些重大的故障导致主分片和复制分片都故障，那这个分片的数据将无法响应给搜索请求。这种情况下，Elasticsearch将报告分片`failed`，但仍将继续返回剩余分片上的结果。

## `timeout`

`time_out`值告诉我们查询超时与否。一般的，搜索请求不会超时。如果响应速度比完整的结果更重要，你可以定义`timeout`参数为`10`或者`10ms`（10毫秒），或者`1s`（1秒）


```javascript
GET /_search?timeout=10ms
```

Elasticsearch将返回在请求超时前收集到的结果。

超时不是一个断路器（circuit breaker）（译者注：关于断路器的理解请看警告）。

> ## 警告

> 需要注意的是`timeout`不会停止执行查询，它仅仅告诉你**目前**顺利返回结果的节点然后关闭连接。在后台，其他分片可能依旧执行查询，尽管结果已经被发送。

> 使用超时是因为对于你的业务需求（译者注：SLA，Service-Level Agreement服务等级协议，在此我翻译为业务需求）来说非常重要，而不是因为你想中断执行长时间运行的查询。

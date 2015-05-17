## geohash单元聚合器

一个查询返回的结果集中可能包含很多的点，以至于不能在地图上全部单独显示。
geohash单元聚合器可以按照你指定的精度计算每个点的geohash并将相邻的点聚合到一起。

返回结果是一个个单元格，每个单元格对应一个可以在地图上展示的 geohash。
通过改变 geohash 的精度，你可以统计全球、某个国家，或者一个城市级别的综述信息。

聚合结果是稀疏（_sparse_）的，因为它只返回包含了文档集合的单元。
如果你的geohash精度太细，导致生成了太多的结果集，它默认只会返回包含结果最多的10000个单元 -- 它们包含了大部分文档集合。
然后，为了找出这排在前10000的单元，它还是需要先生成所有的结果集。
你可以通过如下方式控制生成的单元的数目：

- 1. 使用一个矩形过滤器来限制结果集。
- 2. 对应该矩形，选择一个合适的精度。

```json

GET /attractions/restaurant/_search?search_type=count
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "location": { <1>
            "top_left": {
              "lat":  40,8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.7
            }
          }
        }
      }
    }
  },
  "aggs": {
    "new_york": {
      "geohash_grid": { <2>
        "field":     "location",
        "precision": 5
      }
    }
  }
}
```

- <1> 矩形框将检索限制在纽约区域。
- <2> 使用精度为 `5` 的geohash，精度大约是 5km x 5km.


每个精度为 `5` 的 geohash 覆盖约 25平方公里，那 10000 个单元就能覆盖 25万平方公里。
我们指定的矩形框覆盖面积约 44km * 33km，也就是大概 1452平方公里。
所以这肯定在一个安全的限度内，我们不会因此浪费大量内存来生成太多单元。

上例请求的返回如下：

```json

...
"aggregations": {
  "new_york": {
     "buckets": [ <1>
        {
           "key": "dr5rs",
           "doc_count": 2
        },
        {
           "key": "dr5re",
           "doc_count": 1
        }
     ]
  }
}
...
```

- <1> 每个单元以一个 geohash 作为 `key`。

Again, we didn't specify any subaggregations, so all we got back was the
document count. We could have asked for popular restaurant types, average
price, or other details.
同样的，我们没有指定子聚合器，所以我们的返回结果是文档数目。
我们也可以（指定子聚合器来）得到流行的饭店类型，平均价格，或者其它详细信息。

> 提示

> 为了将这些单元放置在地图上展示，我们需要一个类库来将geohash解析为对于的矩形框或者中心点。
> Javascript和一些语言中有现成的类库，不过你也可以根据 [geo-bounds-agg](/geo-bounds-agg) 的信息自己来实现。

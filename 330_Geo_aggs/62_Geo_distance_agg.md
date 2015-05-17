## 按距离聚合


按距离聚合对于类似“找出距我1公里内的所有pizza店”这样的检索场景很适合。
检索结果需要确实地只返回距离用户1km内的文档，不过我们可以再加上一个“1-2km内的结果集”：

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "query": {
        "match": { <1>
          "name": "pizza"
        }
      },
      "filter": {
        "geo_bounding_box": {
          "location": { <2>
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
    "per_ring": {
      "geo_distance": { <3>
        "field":    "location",
        "unit":     "km",
        "origin": {
          "lat":    40.712,
          "lon":   -73.988
        },
        "ranges": [
          { "from": 0, "to": 1 },
          { "from": 1, "to": 2 }
        ]
      }
    }
  },
  "post_filter": { <4>
    "geo_distance": {
      "distance":   "1km",
      "location": {
        "lat":      40.712,
        "lon":     -73.988
      }
    }
  }
}
```
- <1> 主查询查找饭店名中包含了 “pizza” 的文档。
- <2> 矩形框过滤器让结果集缩小到纽约区域。
- <3> 距离聚合器计算距用户1km和1km-2km的结果数。
- <4> 最后，后置过滤器（`post_filter`)再把结果缩小到距离用户1km的饭店。

上例请求的返回结果如下：

```json

"hits": {
  "total":     1,
  "max_score": 0.15342641,
  "hits": [ <1>
     {
        "_index": "attractions",
        "_type":  "restaurant",
        "_id":    "3",
        "_score": 0.15342641,
        "_source": {
           "name": "Mini Munchies Pizza",
           "location": [
              -73.983,
              40.719
           ]
        }
     }
  ]
},
"aggregations": {
  "per_ring": { <2>
     "buckets": [
        {
           "key":       "*-1.0",
           "from":      0,
           "to":        1,
           "doc_count": 1
        },
        {
           "key":       "1.0-2.0",
           "from":      1,
           "to":        2,
           "doc_count": 1
        }
     ]
  }
}
```

- <1> 后置过滤器（`post_filter`）已经结果集缩小到满足“距离用户1km”条件下的唯一一个pizza店。
- <2> 聚合器包含了"距离用户2km"的pizza店的检索结果。


这个例子中，我们统计了落到各个环形区域中的饭店数。
当然，我们也可以使用子聚合器再在每个环形区域中进一步计算它们的平均价格，最流行，等等。

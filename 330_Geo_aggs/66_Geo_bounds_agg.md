## 范围（边界）聚合器

在geohash聚合器的例子中，我们使用了一个矩形框过滤器来将结果限制在纽约区域。
然而，我们的结果都分布在曼哈顿。
当在地图上呈现给用户时，合理的方式是可以缩放到有数据的区域；地图上有大量空白区域是没有任何点分布的。

范围过滤器是这么做得：
它计算出一个个小矩形框来覆盖到所有的坐标点。

```json

GET /attractions/restaurant/_search?search_type=count
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": {
              "lat":  40,8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.9
            }
          }
        }
      }
    }
  },
  "aggs": {
    "new_york": {
      "geohash_grid": {
        "field":     "location",
        "precision": 5
      }
    },
    "map_zoom": { <1>
      "geo_bounds": {
        "field":     "location"
      }
    }
  }
}
```
- <1> 范围聚合器会计算出一个最小的矩形框来覆盖查询结果的所有文档。

返回结果包含了一个可以用来在地图上缩放的矩形框：

```json

...
"aggregations": {
  "map_zoom": {
     "bounds": {
        "top_left": {
           "lat":  40.722,
           "lon": -74.011
        },
        "bottom_right": {
           "lat":  40.715,
           "lon": -73.983
        }
     }
  },
...
```


实际上，我们可以把矩形聚合器放到每一个 geohash 单元里，因为有坐标点的单元只占了所有单元的一部分：

```json

GET /attractions/restaurant/_search?search_type=count
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": {
              "lat":  40,8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.9
            }
          }
        }
      }
    }
  },
  "aggs": {
    "new_york": {
      "geohash_grid": {
        "field":     "location",
        "precision": 5
      },
      "aggs": {
        "cell": { <1>
          "geo_bounds": {
            "field": "location"
          }
        }
      }
    }
  }
}
```
- <1> 子聚合器 `cell_bounds` 会作用于每个 geohash 单元。


现在落在每个geohash单元中的点都有了一个所在的矩形框区域：

```json

...
"aggregations": {
  "new_york": {
     "buckets": [
        {
           "key": "dr5rs",
           "doc_count": 2,
           "cell": {
              "bounds": {
                 "top_left": {
                    "lat":  40.722,
                    "lon": -73.989
                 },
                 "bottom_right": {
                    "lat":  40.719,
                    "lon": -73.983
                 }
              }
           }
        },
...
```

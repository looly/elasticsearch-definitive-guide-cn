## 按距离排序

检索结果可以按跟指定点的距离排序：

> 提示
> 当你_可以_（_can_）按距离排序时，按距离打分（[scoring-by-distance](#scoring-by-distance)）通常是一个更好的解决方案。

```json
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "type":       "indexed",
          "location": {
            "top_left": {
              "lat":  40,8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.0
            }
          }
        }
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location": { <1>
          "lat":  40.715,
          "lon": -73.998
        },
        "order":         "asc",
        "unit":          "km", <2>
        "distance_type": "plane" <3>
      }
    }
  ]
}
```
- <1> 计算每个文档中 `location` 字段与指定的 `lat/lon` 点间的距离。
- <2> 以 `公里`（`km`）为单位，将距离设置到每个返回结果的 `sort` 键中。
- <3> 使用快速但精度略差的`平面`（`plane`）计算方式。

你可能想问：为什么要制定距离的`单位`（`unit`）呢？
用于排序的话，我们并不关心比较距离的尺度是英里，公里还是光年。
原因是，这个用于排序的值会设置在每个返回结果的 `sort` 元素中。

```json

...
  "hits": [
     {
        "_index": "attractions",
        "_type": "restaurant",
        "_id": "2",
        "_score": null,
        "_source": {
           "name": "New Malaysia",
           "location": {
              "lat": 40.715,
              "lon": -73.997
           }
        },
        "sort": [
           0.08425653647614346 <1>
        ]
     },
...
```
- <1> 宾馆距离我们的指定位置距离是 0.084km。
- 你可以通过设置`单位`（`unit`）来让返回值的形式跟你应用中想要的匹配。

> 提示

> 地理距离排序可以对多个坐标点来使用，不管（这些坐标点）是在文档中还是排序参数中。
> 使用 `sort_mode` 来指定是否需要使用位置集合的 `最小`（`min`），最大（`max`）或者`平均`（`avg`）距离。
> 这样就可以返回``离我的工作地和家最近的朋友``这样的结果了。


### 按距离打分

有可能距离只是决定返回结果排序的唯一重要因素，不过更常见的情况是距离会和其它因素，
比如全文检索匹配度，流行程度或者价格一起决定排序结果。

遇到这种场景你需要在查询分值计算（[`function_score` query](function-score-query)）中指定方式让我们把这些因子处理得到一个综合分。
[decay-functions](decay-functions)中有个一个例子就是地理距离影响排序得分的。

另外按距离排序还有个缺点就是性能：需要对每一个匹配到的文档都进行距离计算。
而 `function_score`请求，在 [`rescore` phase](rescore-api)阶段有可能只需要对前 _n_ 个结果进行计算处理。


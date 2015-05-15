## 地理坐标盒模型过滤器

这是目前为止最有效的 地理坐标过滤器了，因为它计算起来非常简单。
你指定一个矩形的 `顶部`（`top`）, `底部`（`bottom`）, `左边界`（`left`）, 和 `右边界`（`right`），
然后它只需判断坐标的经度是否在左右边界之间，纬度是否在上下边界之间。（译注：原文似乎写反了）

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "location": { <1>
            "top_left": {
              "lat":  40.8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.7,
              "lon": -73.0
            }
          }
        }
      }
    }
  }
}
```
- <1> 盒模型信息也可以用 `bottom_left`（左下方点）和 `top_right`（右上方点） 来表示。

### 优化盒模型

`地理坐标盒模型过滤器`不需要把所有坐标点都加载到内存里。
因为它要做的只是简单判断 `纬度` 和 `经度` 坐标数值是否在给定的范围内，所以它可以用倒排索引来做一个范围（`range`）过滤。

要使用这种优化方式，需要把 `geo_point` 字段用 `纬度`（`lat`）和`经度`（`lon`）方式表示并分别索引。


```json
PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type":    "geo_point",
          "lat_lon": true <1>
        }
      }
    }
  }
}
```
- <1> `location.lat` 和 `location.lon` 字段将被分别索引。它们可以被用于检索，但是不会在检索结果中返回。

然后，查询时你需要告诉 Elasticesearch 使用已索引的 `lat`和`lon`。

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "type":    "indexed", <1>
          "location": {
            "top_left": {
              "lat":  40.8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.7,
              "lon":  -73.0
            }
          }
        }
      }
    }
  }
}
```
- <1> 设置 `type` 参数为 `indexed` (默认为 `memory`) 来明确告诉 Elasticsearch 对这个过滤器使用倒排索引。

> 注意：

> `geo_point` 类型可以包含多个地理坐标点，但是针对经度纬度分别索引的这种优化方式（`lat_lon`）只对单个坐标点的方式有效。



## 地理距离过滤器

`地理距离过滤器`（`geo_distance`）以给定位置为圆心画一个圆，来找出那些位置落在其中的文档：

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance": {
          "distance": "1km", <1>
          "location": { <2>
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```
- <1> 找出所有与指定点距离在1公里（`1km`）内的 `location` 字段。访问 [Distance Units](http://bit.ly/1ynS64j) 查看所支持的距离表示单位
    
- <2> 中心点可以表示为字符串，数组或者（如示例中的）对象。详见 [lat-lon-formats](/lat-lon-formats)。

地理距离过滤器计算代价昂贵。
为了优化性能，Elasticsearch 先画一个矩形框（边长为2倍距离）来围住整个圆形，
这样就可以用消耗较少的盒模型计算方式来排除掉那些不在盒子内（自然也不在圆形内）的文档，
然后只对落在盒模型内的这部分点用地理坐标计算方式处理。

> 提示

> 你需要判断你的使用场景，是否需要如此精确的使用圆模型来做距离过滤？
> 通常使用矩形模型是更高效的方式，并且往往也能满足应用需求。


### 更快的地理距离计算

两点间的距离计算，有多种性能换精度的算法：

- `arc`::

    最慢但是最精确是`弧形`（`arc`）计算方式，这种方式把世界当作是球体来处理。
    不过这种方式精度还是有限，因为这个世界并不是完全的球体。

- `plane`::

   `平面`（`plane`）计算方式，((("plane distance calculation")))把地球当成是平坦的。
    这种方式快一些但是精度略逊；在赤道附近位置精度最好，而靠近两极则变差。

- `sloppy_arc`::

    如此命名，是因为它使用了 Lucene 的 `SloppyMath` 类。
    这是一种用精度换取速度的计算方式，它使用 [Haversine formula](http://en.wikipedia.org/wiki/Haversine_formula) 来计算距离；
    它比`弧形`（`arc`）计算方式快4~5倍, 并且距离精度达99.9%。这也是默认的计算方式。

你可以参考下例来指定不同的计算方式：

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance": {
          "distance":      "1km",
          "distance_type": "plane", <1>
          "location": {
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```
- <1> 使用更快但精度稍差的`平面`（`plane`）计算方式。

> 提示：
> 你的用户真的会在意一个宾馆落在指定圆形区域数米之外了吗？
> 一些地理位置相关的应用会有较高的精度要求；但大部分实际应用场景中，使用精度较低但响应更快的计算方式可能就挺好。

### 地理距离区间过滤器

`地理距离过滤器`（`geo_distance`）和`地理距离区间过滤器`（`geo_distance_range`）的唯一差别在于后者是一个环状的，它会排除掉落在内圈中的那部分文档。


指定到中心点的距离也可以换一种表示方式：
指定一个最小距离（使用 `gt`或者`gte`）和最大距离（使用`lt`或者`lte`），就像使用`区间`（`range`）过滤器一样。

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance_range": {
          "gte":    "1km", <1>
          "lt":     "2km", <1>
          "location": {
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```
- <1> 匹配那些距离中心点超过`1公里`而小于`2公里`的位置。



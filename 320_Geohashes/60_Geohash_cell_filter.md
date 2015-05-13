## geohash单元过滤器

`geohash单元`过滤器做的事情非常简单：
把经纬度坐标位置根据指定精度转换成一个geohash，然后查找落在同一个geohash中的位置--这实在是非常高效的过滤器。

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geohash_cell": {
          "location": {
            "lat":  40.718,
            "lon": -73.983
          },
          "precision": "2km" <1>
        }
      }
    }
  }
}
```
- <1> `precision` 字段设置的精度不能高于geohash精度映射时的设定。

这个过滤器将坐标点转换成对应长度的geohash--本例中为`dr5rsk`--然后查找位于同一个组中的所有位置。

然而，如上例中的写法可能不会返回5km内所有的宾馆。
要知道每个 geohash 实际上仅是一个矩形，而指定的点可能位于这个矩形中的任何位置。
有可能这个点刚好落在了geohash单元的边缘附近，但过滤器会排除那些（挨得很近却）落在相邻单元里的宾馆。

为了修正这点，我们可以告诉过滤器，把周围的单元也包含进来。
通过设置`neighbors` 参数为 `true`：

```json

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geohash_cell": {
          "location": {
            "lat":  40.718,
            "lon": -73.983
          },
          "neighbors": true, <1>
          "precision": "2km"
        }
      }
    }
  }
}
```

- <1> 过滤器将会查找对应的geohash和包围它的（8个）geohash。

明显的，`2km`精度的geohash再加上周围的单元，会导致结果实际在一个更大的检索范围。
这个过滤器不是为精度而生的，但是它非常有效率，可以用于更高精度的地理位置过滤器的前置过滤器。

> 提示

> 将 `precision` 参数设置为一个距离可能会有误导性。
> 比如将 `precision` 设置为 `2km` 将会转换成长度为6的geohash。但实际上它的尺寸是约 1.2km * 0.6 km。
> 你可能会发现这还不如自己明确的设置一个长度 `5` 或者 `6` 来得更容易理解。

这个过滤器有一个比`地理盒模型过滤器`（`geo_bounding_box`）更好的优点，就是它支持一个字段中有多个坐标位置的情况。
我们在设置优化盒模型过滤器（ [optimize-bounding-box](optimize-bounding-box) ）讲过，设置 `lat_lon` 选项也是一个很有效的方式，
但是它只对字段中的单个坐标点情况有效。



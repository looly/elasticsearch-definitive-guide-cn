## 地理形状的过滤与缓存

`地理形状` 的查询和过滤都是表现为相同功能。查询就是简单的表现为一个过滤：把所以匹配到的文档的 `_score` 标记为1。
查询结果不能被缓存，不过过滤结果可以。

结果默认是不被缓存的。与地理坐标点集类似，任何形状内坐标的变化都会导致 geohash 集合的变化，因此在缓存过滤结果几乎没有什么意义。
也就是说，除非你会重复的使用相同的形状来做过滤，它才是值得缓存起来的。
缓存方法是，把 `_cache` 设置为 `true`：

```json

GET /attractions/neighborhood/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_shape": {
          "_cache": true, <1>
          "location": {
            "indexed_shape": {
              "index": "attractions",
              "type":  "landmark",
              "id":    "dam_square",
              "path":  "location"
            }
          }
        }
      }
    }
  }
}
```
 - <1> `geo_shape` 过滤器的结果将被缓存。


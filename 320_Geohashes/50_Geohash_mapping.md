## Geohashes 映射

首先，你需要确定你需要什么样的精度。
虽然你也可以使用12级的精度来索引所有的地理坐标点，但是你真的需要精确到数厘米的精度吗？
如果你把精度控制在一个实际一些的值，比如 `1km`，那么你可以节省大量的索引空间：

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
          "type":               "geo_point",
          "geohash_prefix":     true, <1>
          "geohash_precision":  "1km" <2>
        }
      }
    }
  }
}
```
- <1> 将 `geohash_prefix` 设为 `true` 来告诉 Elasticsearch 使用指定精度来做 geohash 前缀索引。
- <2> 精度描述可以是一个具体数字，表示 geohash 的长度，也可以是一个距离。精度设为 `1km`表示geohash长度是 `7`。

通过如上设置，geohash前缀为 1-7 的部分将被索引，所能提供的精度大约在150米。

## 减少内存占用

每一个 `经纬度`（`lat/lon`）组合需要占用16个字节的内存。要知道内存可是供不应求的。
使用这种占用16字节内存的方式可以得到非常精确的结果。不过就像之前提到的一样，实际应用中几乎都不需要这么精确。

你可以通过这种方式来减少内存使用量：
设置一个`压缩的`（`compressed`）数据字段格式并明确指定你的地理坐标点所需的精度。
即使只是将精度降低到`1毫米`（`1mm`）级别，也可以减少1/3的内存使用。
更实际的，将精度设置到`3米`（`3m`）内存占用可以减少62%，而设置到`1公里`（`1km`）则节省75%之多。

这个设置项可以通过 `update-mapping` API 来对实时索引进行调整：

```json
POST /attractions/_mapping/restaurant
{
  "location": {
    "type": "geo_point",
    "fielddata": {
      "format":    "compressed",
      "precision": "1km" <1>
    }
  }
}
```
- <1> 每一个`经纬度`（`lat/lon`）组合现在只需要4个字节，而不是16个。

另外，你还可以这样做来避免把所有地理坐标点全部同时加载到内存中：
使用在优化盒模型（[optimize-bounding-box](optimize-bounding-box)）中提到的技术，
或者把地理坐标点当作文档值（[doc values](doc-values)）来存储。

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
          "type":       "geo_point",
          "doc_values": true <1>
        }
      }
    }
  }
}
```
- <1> 地理坐标点现在不会被加载到内存，而是保存在磁盘中。


将地理坐标点映射为文档值的方式只能是在这个字段第一次被创建时。
相比使用字段值，使用文档值会有一些小的性能代价，不过考虑到它对内存的节省，这种方式通常是还值得的。

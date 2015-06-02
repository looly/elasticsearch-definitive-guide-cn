## 映射地理形状

与 `geo_point`类型的字段相似，地理形状也需要在使用前明确映射：

```json

PUT /attractions
{
  "mappings": {
    "landmark": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type": "geo_shape"
        }
      }
    }
  }
}
```

你需要关注两个重要的设置项来调整精度（`precision`）和距离误差（`distance_error_pct`）。
There are two important settings that you should consider changing `precision` and `distance_error_pct`.

### 精度

精度（`precision`）参数用来控制组成地理形状的geohash的长度。
它的默认值是 `9`，等同于尺寸在 5m*5m 的[geohash](geohash)。这个精度可能比你需要的精确得多。

精度越低，需要索引的单元就越少，检索时也会更快。当然，精度越低，地理形状的准确性就越差。
你需要决定自己的地理形状所需精度 —— 即使减少1-2个等级的精度也能带来明显的消耗缩减收益。

你可以通过指定距离的方式来定制精度 —— 比如，`50m`或`2km`；
不过这些距离最终也是会转换成对应的geohash长度（见 [geohashes](geohashes)）。


### 距离误差

当索引一个多边形时，中间连续区域很容易用一个短geohash来表示。
麻烦的是边缘部分，这些地方需要使用更精细的geohash才能表示。

当你在索引一个小地标时，你希望它的边界比较精确；（如果精度不够，）让这些纪念碑一个叠着一个可不好。
当索引整个国家时，你就不需要这么高的精度。误差个50米左右也没什么大不了。


距离误差（`distance_error_pct`）指定地理形状可以接受的最大错误率。它的默认值是 `0.025`，即 2.5%。
这也就是说，大的地理形状（比如国家）相比小的地理形状（比如纪念碑）来说，容许更加模糊的边界。

`0.025`是一个不错的初始值。不过如果我们容许更大的错误率，对应地理形状需要索引的单元就越少。

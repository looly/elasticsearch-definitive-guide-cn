## 地理坐标点

_地理坐标点（geo-point）_ 是指地球表面可以用经纬度描述的一个点。地理坐标点可以用来计算两个坐标位置间的距离，或者判断一个点是否在一个区域中。

地理坐标点不能被动态映射（dynamic mapping）自动检测，而是需要显式声明对应字段类型为 `geo_point` 。

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
          "type": "geo_point"
        }
      }
    }
  }
}
```

### 经纬度坐标格式

如上例，`location` 被声明为 `geo_point` 后，我们就可以索引包含了经纬度信息的文档了。经纬度信息的形式可以是字符串，数组或者对象。

```json
PUT /attractions/restaurant/1
{
  "name":     "Chipotle Mexican Grill",
  "location": "40.715, -74.011" <1>
}

PUT /attractions/restaurant/2
{
  "name":     "Pala Pizza",
  "location": { <2>
    "lat":     40.722,
    "lon":    -73.989
  }
}

PUT /attractions/restaurant/3
{
  "name":     "Mini Munchies Pizza",
  "location": [ -73.983, 40.719 ] <3>
}
```
- <1> 以半角逗号分割的字符串形式 `"lat,lon"`；
- <2> 明确以 `lat` 和 `lon` 作为属性的对象；
- <3> 数组形式表示 `[lon,lat]`。


> 注意

> 可能所有人都至少踩过一次这个坑：地理坐标点用字符串形式表示时是纬度在前，经度在后（`"latitude,longitude"`），而数组形式表示时刚好相反，是经度在前，纬度在后（`[longitude,latitude]`）。

> 其实，在 Elasticesearch 内部，不管字符串形式还是数组形式，都是纬度在前，经度在后。不过早期为了适配 GeoJSON 的格式规范，调整了数组形式的表示方式。

> 因此，在使用地理位置（geolocation）的路上就出现了这么一个“捕熊器”，专坑那些不了解这个陷阱的使用者。

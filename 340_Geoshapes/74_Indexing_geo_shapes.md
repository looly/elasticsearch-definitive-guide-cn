## 索引地理形状

地理形状通过[GeoJSON](http://geojson.org)来表示，这是一种开放的使用JSON实现的二维形状编码方式。
每个形状包含两个信息：形状类型：`point`, `line`, `polygon`, `envelope`；一个或多经纬度点集合的数组。


> 注意：
> 
> 在 GeoJSON 里，经纬度表示方式通常是“纬度在前，经度在后”。


举例如下，我们用一个多边形来索引阿姆斯特丹达姆广场：

```json

PUT /attractions/landmark/dam_square
{
    "name" : "Dam Square, Amsterdam",
    "location" : {
        "type" : "polygon", <1>
        "coordinates" : [[ <2>
          [ 4.89218, 52.37356 ],
          [ 4.89205, 52.37276 ],
          [ 4.89301, 52.37274 ],
          [ 4.89392, 52.37250 ],
          [ 4.89431, 52.37287 ],
          [ 4.89331, 52.37346 ],
          [ 4.89305, 52.37326 ],
          [ 4.89218, 52.37356 ]
        ]]
    }
}
```

- <1> `type`参数指明如何使用经纬度坐标集来表示对应形状。
- <2> 用来表示多边形的经纬度坐标点列表。

上例中大量的方括号可能看起来让人困惑，不过实际上 GeoJSON 的语法非常简单：

1. 用一个数组表示经纬度坐标点:
```json
    [lon,lat]
```

2. 一组坐标点放到一个数组来表示一个多边形：
```json
    [[lon,lat],[lon,lat], ... ]
```

3. 一个多边形（`polygon`）形状可以包含多个多边形；第一个表示多边形的外轮廓，后续的多边形表示第一个多边形内部的空洞：
```json
    [
      [[lon,lat],[lon,lat], ... ],  # main polygon
      [[lon,lat],[lon,lat], ... ],  # hole in main polygon
      ...
    ]
```

参见 [Geo-shape mapping documentation](http://bit.ly/1G2nMCT) 了解更多支持的形状。


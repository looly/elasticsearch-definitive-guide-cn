## Geohashes

[Geohashes](http://en.wikipedia.org/wiki/Geohash) 是一种将 经纬度坐标对（`lat/lon`）编码成字符串的方式。
最开始这么做只是为了让地理位置在url上呈现的形式更加友好，
不过现在geohash已经变成一种在数据库中有效索引地理坐标点和地理形状的方式。

Geohashes 把整个世界分为32个单元的格子--4行8列--每一个格子都用一个字母或者数字标识。
比如 `g` 这个单元覆盖了半个格林兰，冰岛的全部和大不列颠的大部分。
每一个单元还可以进一步被分解成新的32个单元，这些单元又可以继续被分解成32个更小的单元，不断重复下去。
`gc` 这个单元覆盖了爱尔兰和英格兰，`gcp`覆盖了伦敦的大部分和部分南英格兰，
`gcpuuz94k`是伯明翰宫的入口，精确到了约5米。

换句话说，geohash的长度越长，它的精度就越高。
如果两个geohash有一个共同的前缀，如 `gcpuuz`，就表示他们挨得很紧。
共同的前缀越长，距离就越近。

但那也就是说，两个刚好相邻的位置，会可能有完全不同的geohash。
一个实例，伦敦的 [Millenium Dome](http://en.wikipedia.org/wiki/Millennium_Dome) 的geohash是 `u10hbp`，
因为它落在了 `u`这个大单元里，而紧挨着它东边的最大的单元是 `g`。

地理坐标点可以自动关联到他们对应的 geohash。
需要注意的是，他们会被索引到了所有（各个层级）的 geohash _前缀_（_prefixes_）。
例：索引伯明翰宫的门口--坐标纬度 `51.501568`，经度`-0.141257`--会在各种尺寸精度的 geohash 上建立索引，
如下表：

Geohash        |Level| Dimensions
---------------|-----|---------------------
g              |1    | ~ 5,004km x 5,004km
gc             |2    | ~ 1,251km x 625km
gcp            |3    | ~ 156km x 156km
gcpu           |4    | ~ 39km x 19.5km
gcpuu          |5    | ~ 4.9km x 4.9km
gcpuuz         |6    | ~ 1.2km x 0.61km
gcpuuz9        |7    | ~ 152.8m x 152.8m
gcpuuz94       |8    | ~ 38.2m x 19.1m
gcpuuz94k      |9    | ~ 4.78m x 4.78m
gcpuuz94kk     |10   | ~ 1.19m x 0.60m
gcpuuz94kkp    |11   | ~ 14.9cm x 14.9cm
gcpuuz94kkp5   |12   | ~ 3.7cm x 1.8cm

geohash单元过滤器（ [`geohash_cell` filter](http://bit.ly/1DIqyex) ）可以使用这些geohash前缀来找出与指定坐标点（`lat/lon`）相邻的位置。

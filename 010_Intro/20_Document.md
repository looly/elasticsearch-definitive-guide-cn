=== Document oriented
## 面向文档

程序中的对象很少是简单的键值列表。更多时候它拥有复杂数据结构，比如包含日期、地理位置、对象或者数组。

迟早你想会把这些对象存储到数据库中。尝试将这些数据保存到由行和列组成的关系数据库中，就好像要把一个复杂对象放入一个非常大的表格中：你不得不调整对象以适应表模式（通常一列表示一个字段）然后不得不在检索的时候重建对象。

Elasticsearch是**面向文档(document oriented)**的，这意味着它可以存储整个对象或**文档(document)**，它不仅是存储，还会**索引(index)**每个文档的内容使之可以被搜索。在Elasticsearch中你可以对文档索引、搜索、排序、过滤，而非成行成列的数据。这种理解数据的方式与以往完全不同，这也是Elasticsearch能够执行复杂的全文搜索的原因之一。

## JSON
ELasticsearch使用[**JSON**](http://en.wikipedia.org/wiki/Json)（或者叫做**Javascript对象符号(JavaScript
Object Notation)**）做为文档序列化格式。JSON现在已经被大多语言所支持，而且已经成为NoSQL领域的标准格式。它简单、简洁且容易被阅读。
以下这个JSON文档表示一个用户对象：

```Javascript
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "info": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01"
}
```

尽管原始的`user`对象很复杂，但它的结构和对象的含义已经被保留在JSON中了，转换对象为JSON并作为索引要比在表结构中做相同事情简单多了。

>转换你的数据为JSON
>几乎所有语言都有相应的模块用于将任意数据结构转换为JSON，但每种语言处理细节不同。具体请查看“`serialization`” or “`marshalling`”两个用于处理JSON的模块。[Elasticsearch官方客户端](http://www.elasticsearch.org/guide)会自动为你序列化和反序列化JSON。

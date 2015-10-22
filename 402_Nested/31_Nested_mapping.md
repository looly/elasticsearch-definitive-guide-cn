[[巢状-映射]]
=== 巢状对象映射

设定一个`nested`栏位很简单--在((("mapping (types)", "nested object")))((("nested object mapping")))你会设定为`object`类型的地方，改为`nested`类型：

[source,json]
--------------------------
PUT /my_index
{
  "mappings": {
    "blogpost": {
      "properties": {
        "comments": {
          "type": "nested", <1>
          "properties": {
            "name":    { "type": "string"  },
            "comment": { "type": "string"  },
            "age":     { "type": "short"   },
            "stars":   { "type": "short"   },
            "date":    { "type": "date"    }
          }
        }
      }
    }
  }
}
--------------------------
<1> 一个`nested`栏位接受与`object`类型相同的参数。

所需仅此而已。 任何`comments`对象会被索引为分离巢状对象。
参考更多 http://bit.ly/1KNQEP9[`nested` type reference docs]。


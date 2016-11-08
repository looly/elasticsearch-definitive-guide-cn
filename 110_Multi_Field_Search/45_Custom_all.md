#### 自定义_all字段

在[元数据：_all字段](../070_Index_Mgmt/32_Metadata_all.md)中，我们解释了特殊的_all字段会将其它所有字段中的值作为一个大字符串进行索引。尽管将所有字段的值作为一个字段进行索引并不是非常灵活。如果有一个自定义的_all字段用来索引人名，另外一个自定义的_all字段用来索引地址就更好了。

ES通过字段映射中的copy_to参数向我们提供了这一功能：

```Javascript
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name" <1>
                },
                "last_name": {
                    "type":     "string",
                    "copy_to":  "full_name" <1>
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/45_Custom_all.json

<1> first_name和last_name字段中的值会被拷贝到full_name字段中。

有了这个映射，我们可以通过first_name字段查询名字，last_name字段查询姓氏，或者full_name字段查询姓氏和名字。

> 提示：first_name和last_name字段的映射和full_name字段的索引方式的无关。full_name字段会从其它两个字段中拷贝字符串的值，然后仅根据full_name字段自身的映射进行索引。


<!--
[[custom-all]]
=== Custom _all Fields

In <<all-field>>, we explained that the special `_all` field indexes the values
from all other fields as one big string.((("_all field", sortas="all field")))((("multifield search", "custom _all fields"))) Having all fields indexed into one
field is not terribly flexible, though.  It would be nice to have one custom
`_all` field for the person's name, and another custom `_all` field for the
address.

Elasticsearch provides us with this functionality via the `copy_to` parameter
in a field ((("copy_to parameter")))((("mapping (types)", "copy_to parameter")))mapping:

[source,js]
--------------------------------------------------
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name" <1>
                },
                "last_name": {
                    "type":     "string",
                    "copy_to":  "full_name" <1>
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/45_Custom_all.json

<1> The values in the `first_name` and `last_name` fields
    are also copied to the `full_name` field.

With this mapping in place, we can query the `first_name` field for first
names, the `last_name` field for last name, or the `full_name` field for first
and last names.

NOTE: Mappings of the `first_name` and `last_name` fields have no bearing
on how the `full_name` field is indexed. The `full_name` field copies the
string values from the other two fields, then indexes them according to the
mapping of the `full_name` field only.

-->
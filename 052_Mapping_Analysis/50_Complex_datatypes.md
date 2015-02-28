## 复合核心字段类型

除了之前提到的简单的标量类型，JSON还有`null`值，数组和对象，所有这些Elasticsearch都支持：

### 多值字段

我们想让`tag`字段包含多个字段，这非常有可能发生。我们可以索引一个标签数组来代替单一字符串：

```javascript
{ "tag": [ "search", "nosql" ]}
```

对于数组不需要特殊的映射。任何一个字段可以包含零个、一个或多个值，同样对于全文字段被分析产生多个词。

言外之意，这意味着**数组中所有值必须为同一类型**。你不能把日期和字符窜混合。如果你创建一个新字段，这个字段索引了一个数组，Elasticsearch将使用第一个值的类型来确定这个新字段的类型。

****

When you get a document back from Elasticsearch, any arrays will be in the
same order as when you indexed the document.  The `_source` field that you get
back contains exactly the same JSON document that you indexed.

However, arrays are _indexed_ -- made searchable -- as multi-value fields,
which are unordered.  At search time you can't refer to ``the first element''
or ``the last element''.  Rather think of an array as a _bag of values_.

****

==== Empty fields

Arrays can, of course, be empty. This is the equivalent of having zero
values. In fact, there is no way of storing a `null` value in Lucene, so
a field with a `null` value is also considered to be an empty
field.

These four fields would all be considered to be empty, and would not be
indexed:

[source,js]
--------------------------------------------------
"empty_string":             "",
"null_value":               null,
"empty_array":              [],
"array_with_null_value":    [ null ]
--------------------------------------------------

[[inner-objects]]
==== Multi-level objects

The last native JSON datatype that we need to discuss is the _object_
-- known in other languages as hashes, hashmaps, dictionaries or
associative arrays.

_Inner objects_ are often used to embed one entity or object inside
another. For instance, instead of having fields called `user_name`
and `user_id` inside our `tweet` document, we could write it as:

[source,js]
--------------------------------------------------
{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}
--------------------------------------------------


==== Mapping for inner objects

Elasticsearch will detect new object fields dynamically and map them as
type `object`, with each inner field listed under `properties`:

[source,js]
--------------------------------------------------
{
  "gb": {
    "tweet": { <1>
      "properties": {
        "tweet":            { "type": "string" },
        "user": { <2>
          "type":             "object",
          "properties": {
            "id":           { "type": "string" },
            "gender":       { "type": "string" },
            "age":          { "type": "long"   },
            "name":   { <2>
              "type":         "object",
              "properties": {
                "full":     { "type": "string" },
                "first":    { "type": "string" },
                "last":     { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
--------------------------------------------------
<1> Root object.
<2> Inner objects.

The mapping for the `user` and `name` fields have a similar structure
to the mapping for the `tweet` type itself.  In fact, the `type` mapping
is just a special type of `object` mapping, which we refer to as the
_root object_.  It is just the same as any other object, except that it has
some special top-level fields for document metadata, like `_source`,
the `_all` field etc.

==== How inner objects are indexed

Lucene doesn't understand inner objects. A Lucene document consists of a flat
list of key-value pairs.  In order for Elasticsearch to index inner objects
usefully, it converts our document into something like this:

[source,js]
--------------------------------------------------
{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
--------------------------------------------------


_Inner fields_ can be referred to by name, eg `"first"`. To distinguish
between two fields that have the same name we can use the full _path_,
eg `"user.name.first"` or even the `type` name plus
the path: `"tweet.user.name.first"`.

NOTE: In the simple flattened document above, there is no field called `user`
and no field called `user.name`.  Lucene only indexes scalar or simple values,
not complex datastructures.

[[object-arrays]]
==== Arrays of inner objects

Finally, consider how an array containing inner objects would be indexed.
Let's say we have a `followers` array which looks like this:

[source,js]
--------------------------------------------------
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
--------------------------------------------------


This document will be flattened as we described above, but the result will
look like this:

[source,js]
--------------------------------------------------
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
--------------------------------------------------


The correlation between `{age: 35}` and `{name: Mary White}` has been lost as
each multi-value field is just a bag of values, not an ordered array.  This is
sufficient for us to ask:

* _Is there a follower who is 26 years old?_

but we can't get an accurate answer to:

* _Is there a follower who is 26 years old **and who is called Alex Jones?**_

Correlated inner objects, which are able to answer queries like these,
are called _nested_ objects, and we will discuss them later on in
<<nested-objects>>.


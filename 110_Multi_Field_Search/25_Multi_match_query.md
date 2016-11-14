### multi_match查询

multi_match查询提供了一个简便的方法用来对多个字段执行相同的查询。

> 提示：存在几种类型的multi_match查询，其中的3种正好和在["单一查询字符串"小节中"了解你的数据"单元](../110_Multi_Field_Search/10_Single_query_string.md)中提到的几种类型相同：best_fields，most_fields以及cross_fields。

默认情况下，该查询以best_fields类型执行，它会为每个字段生成一个match查询，然后将这些查询包含在一个dis_max查询中。下面的dis_max查询：

```Javascript
{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}
```

可以通过multi_match简单地重写如下：

```Javascript
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", <1>
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" <2>
    }
}
```
// SENSE: 110_Multi_Field_Search/25_Best_fields.json

<1> 注意到以上的type属性为best_fields。
<2> minimum_should_match和operator参数会被传入到生成的match查询中。

#### 在字段名中使用通配符

字段名可以通过通配符指定：任何匹配了通配符的字段都会被包含在搜索中。你可以通过下面的查询来匹配book_title，chapter_title以及section_title字段：

```Javascript
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
```
#### 加权个别字段

个别字段可以通过caret语法(^)进行加权：仅需要在字段名后添加^boost，其中的boost是一个浮点数：

```Javascript
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] <1>
    }
}
```

<1> chapter_title字段的boost值为2，而book_title和section_title字段的boost值为默认的1。





<!-- 
[[multi-match-query]]
=== multi_match Query

The `multi_match` query provides ((("multifield search", "multi_match query")))((("multi_match queries")))((("match query", "multi_match queries"))) a convenient shorthand way of running
the same query against multiple fields.

[NOTE]
====
There are several types of `multi_match` query, three of which just
happen to coincide with the three scenarios that we listed in
<<know-your-data>>:  `best_fields`, `most_fields`, and `cross_fields`.
====

By default, this query runs as type `best_fields`, which means((("best fields queries", "multi-match queries")))((("dis_max (disjunction max) query", "multi_match query wrapped in"))) that it generates a
`match` query for each field and wraps them in a `dis_max` query. This
`dis_max` query

[source,js]
--------------------------------------------------
{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}
--------------------------------------------------

could be rewritten more concisely with `multi_match` as follows:

[source,js]
--------------------------------------------------
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", <1>
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" <2>
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/25_Best_fields.json

<1> The `best_fields` type is the default and can be left out.
<2> Parameters like `minimum_should_match` or `operator` are passed through to
    the generated `match` queries.

==== Using Wildcards in Field Names

Field names can be specified with wildcards: any field that matches the
wildcard pattern((("multi_match queries", "wildcards in field names")))((("wildcards in field names")))((("fields", "wildcards in field names"))) will be included in the search. You could match on the
`book_title`, `chapter_title`, and `section_title` fields, with the following:

[source,js]
--------------------------------------------------
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
--------------------------------------------------

==== Boosting Individual Fields

Individual fields can be boosted by using the caret (`^`) syntax: just add
`^boost` after the field((("multi_match queries", "boosting individual fields")))((("boost parameter", "boosting individual fields in multi_match queries"))) name, where `boost` is a floating-point number:

[source,js]
--------------------------------------------------
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] <1>
    }
}
--------------------------------------------------

<1> The `chapter_title` field has a `boost` of `2`, while the `book_title` and
    `section_title` fields have a default boost of `1`.

-->
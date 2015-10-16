[[巢状-查询]]
=== 查询巢状对象

因巢状对象((("nested objects", "querying")))会被索引为分离隐藏文档，我们不能直接查询它们。
((("queries", "nested"))) 而是使用 http://bit.ly/1ziFQoR[`nested`查询]或 http://bit.ly/1IOp94r[`nested` 过滤器]来存取它们：

[source,json]
--------------------------
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "eggs" }}, <1>
        {
          "nested": {
            "path": "comments", <2>
            "query": {
              "bool": {
                "must": [ <3>
                  { "match": { "comments.name": "john" }},
                  { "match": { "comments.age":  28     }}
                ]
        }}}}
      ]
}}}
--------------------------
<1> `title`条件运作在根文档上
<2> `nested`条件``深入``巢状的`comments`栏位。
    它不会在存取根文档的栏位，或是其他巢状文档的栏位。
<3> `comments.name`以及`comments.age`运作在相同的巢状文档。

[TIP]
==================================================

一个`nested`栏位可以包含其他`nested`栏位。 相同的，一个`nested`查询可以包含其他`nested`查询。
巢状阶层会如同你预期的运作。

==================================================

当然，一个`nested`查询可以匹配多个巢状文档。
每个文档的匹配会有各自的关联分数，但多个分数必须减少至单一分数才能应用至根文档。

在预设中，它会平均所有巢状文档匹配的分数。这可以藉由设定`score_mode`参数为`avg`, `max`, `sum`或甚至`none`(为了防止根文档永远获得`1.0`的匹配分数时)来控制。

[source,json]
--------------------------
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "eggs" }},
        {
          "nested": {
            "path":       "comments",
            "score_mode": "max", <1>
            "query": {
              "bool": {
                "must": [
                  { "match": { "comments.name": "john" }},
                  { "match": { "comments.age":  28     }}
                ]
        }}}}
      ]
}}}
--------------------------
<1> 从最匹配的巢状文档中给予根文档的`_score`值。

[注意]
====
`nested`过滤器类似於`nested`查询，除了无法使用`score_mode`参数。 只能使用在_filter context_&#x2014;例如在`filtered`查询中--其作用类似其他的过滤器：
包含或不包含，但不评分。

`nested`过滤器的结果本身不会缓存，通常缓存规则会被应用於`nested`过滤器_之中_的过滤器。
====


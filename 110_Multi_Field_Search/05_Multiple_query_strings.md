<!--秀川译-->
### 多重查询字符串

在明确的字段中的词查询是最容易处理的多字段查询。如果我们知道War and Peace是标题，Leo Tolstoy是作者，可以很容易的用match查询表达每个条件，并且用布尔查询组合起来：

```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }}
      ]
    }
  }
}
```

布尔查询采用"匹配越多越好(More-matches-is-better)"的方法，所以每个match子句的得分会被加起来变成最后的每个文档的得分。匹配两个子句的文档的得分会比只匹配了一个文档的得分高。

当然，没有限制你只能使用match子句：布尔查询可以包装任何其他的查询类型，包含其他的布尔查询，我们可以添加一个子句来指定我们更喜欢看被哪个特殊的翻译者翻译的那版书：

```Javascript
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```
为什么我们把翻译者的子句放在一个独立的布尔查询中？所有的匹配查询都是should子句，所以为什么不把翻译者的子句放在和title以及作者的同一级？

答案就在如何计算得分中。布尔查询执行每个匹配查询，把他们的得分加在一起，然后乘以匹配子句的数量，并且除以子句的总数。每个同级的子句权重是相同的。在前面的查询中，包含翻译者的布尔查询占用总得分的三分之一。如果我们把翻译者的子句放在和标题与作者同级的目录中，我们会把标题与作者的作用减少的四分之一。

### 设置子句优先级

在先前的查询中我们可能不需要使每个子句都占用三分之一的权重。我们可能对标题以及作者比翻译者更感兴趣。我们需要调整查询来使得标题和作者的子句相关性更重要。

最简单的方法是使用boost参数。为了提高标题和作者字段的权重，我们给boost参数提供一个比1高的值：

```Javascript
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { <1>
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { <1>
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { <2>
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```

<1> 标题和作者的boost值为2。
<2> 嵌套的布尔查询的boost值为默认的1。

通过试错(Trial and Error)的方式可以确定"最佳"的boost值：设置一个boost值，执行测试查询，重复这个过程。一个合理boost值的范围在1和10之间，也可能是15。比它更高的值的影响不会起到很大的作用，因为分值会[被规范化(Normalized)](https://www.elastic.co/guide/en/elasticsearch/guide/current/_boosting_query_clauses.html#boost-normalization)。

<!--
[[multi-query-strings]]
=== Multiple Query Strings

The simplest multifield query to deal with is the ((("multifield search", "multiple query strings")))one where we can _map
search terms to specific fields_. If we know that _War and Peace_ is the
title, and Leo Tolstoy is the author, it is easy to write each of these
conditions as a `match` clause ((("match clause, mapping search terms to specific fields")))((("bool query", "mapping search terms to specific fields in match clause")))and to combine them with a <<bool-query,`bool`
query>>:

[source,js]
--------------------------------------------------
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }}
      ]
    }
  }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/05_Multiple_query_strings.json

The `bool` query takes a _more-matches-is-better_ approach, so the score from
each `match` clause will be added together to provide the final `_score` for
each document. Documents that match both clauses will score higher than
documents that match just one clause.

Of course, you're not restricted to using just `match` clauses: the `bool`
query can wrap any other query type, ((("bool query", "nested bool query in")))including other `bool` queries.  We could
add a clause to specify that we prefer to see versions of the book that have
been translated by specific translators:

[source,js]
--------------------------------------------------
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/05_Multiple_query_strings.json


Why did we put the translator clauses inside a separate `bool` query?  All four
`match` queries are `should` clauses, so why didn't we just put the translator
clauses at the same level as the title and author clauses?

The answer lies in how the score is calculated.((("relevance scores", "calculation in bool queries")))  The `bool` query runs each
`match` query, adds their scores together, then multiplies by the number of
matching clauses, and divides by the total number of clauses. Each clause at
the same level has the same weight. In the preceding query, the `bool` query
containing the translator clauses counts for one-third of the total score. If we had
put the translator clauses at the same level as title and author, they
would have reduced the contribution of the title and author clauses to one-quarter each.

[[prioritising-clauses]]
==== Prioritizing Clauses

It is likely that an even one-third split between clauses is not what we need for
the preceding query. ((("multifield search", "multiple query strings", "prioritizing query clauses")))((("bool query", "prioritizing clauses"))) Probably we're more interested in the title and author
clauses then we are in the translator clauses. We need to tune the query to
make the title and author clauses relatively more important.

The simplest weapon in our tuning arsenal is the `boost` parameter. To
increase the weight of the `title` and `author` fields, give ((("boost parameter", "using to prioritize query clauses")))((("weight", "using boost parameter to prioritize query clauses")))them a `boost`
value higher than `1`:

[source,js]
--------------------------------------------------
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { <1>
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { <1>
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { <2>
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/05_Multiple_query_strings.json

<1> The `title` and `author` clauses have a `boost` value of `2`.
<2> The nested `bool` clause has the default `boost` of `1`.

The ``best'' value for the `boost` parameter is most easily determined by
trial and error: set a `boost` value, run test queries, repeat. A reasonable
range for `boost` lies between `1` and `10`, maybe `15`. Boosts higher than
that have little more impact because scores are
<<boost-normalization,normalized>>.

-->

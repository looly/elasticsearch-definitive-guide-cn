<!-- 秀川 -->
### 组合查询

在《组合过滤》中我们讨论了怎样用布尔过滤器组合多个用`and`, `or`, and `not`逻辑组成的过滤子句，在查询中, 布尔查询充当着相似的作用，但是有一个重要的区别。

过滤器会做一个判断: 是否应该将文档添加到结果集? 然而查询会做更精细的判断. 他们不仅决定一个文档是否要添加到结果集，而且还要计算文档的相关性（_relevant_）.

像过滤器一样, 布尔查询接受多个用`must`, `must_not`, and `should`的查询子句.  例:


```Javascript
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "quick" }},
      "must_not": { "match": { "title": "lazy"  }},
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ]
    }
  }
}
```

在前面的查询中，凡是满足`title`字段中包含`quick`，但是不包含`lazy`的文档都会在查询结果中。到目前为止，布尔查询的作用非常类似于布尔过滤的作用。

当`should`过滤器中有两个子句时不同的地方就体现出来了，下面例子就可以体现：一个文档不需要同时包含`brown`和`dog`，但如果同时有这两个词，这个文档的相关性就更高:

```Javascript
{
  "hits": [
     {
        "_id":      "3",
        "_score":   0.70134366, <1>
        "_source": {
           "title": "The quick brown fox jumps over the quick dog"
        }
     },
     {
        "_id":      "1",
        "_score":   0.3312608,
        "_source": {
           "title": "The quick brown fox"
        }
     }
  ]
}
```

<1> 文档3的得分更高，是因为它同时包含了`brown` 和 `dog`。

####得分计算
布尔查询通过把所有符合`must` 和 `should`的子句得分加起来，然后除以`must` 和 `should`子句的总数为每个文档计算相关性得分。

`must_not`子句并不影响得分；他们存在的意义是排除已经被包含的文档。


#### 精度控制

所有的 `must` 子句必须匹配, 并且所有的 `must_not` 子句必须不匹配, 但是多少 `should` 子句应该匹配呢? 默认的，不需要匹配任何 `should` 子句，一种情况例外：如果没有`must`子句，就必须至少匹配一个`should`子句。

像我们控制`match`查询的精度一样，我们也可以通过`minimum_should_match`参数控制多少`should`子句需要被匹配，这个参数可以是正整数，也可以是百分比。

```Javascript
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "brown" }},
        { "match": { "title": "fox"   }},
        { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 <1>
    }
  }
}
```

<1> 这也可以用百分比表示

结果集仅包含`title`字段中有`"brown"
和 "fox"`, `"brown" 和 "dog"`, 或 `"fox" 和 "dog"`的文档。如果一个文档包含上述三个条件，那么它的相关性就会比其他仅包含三者中的两个条件的文档要高。

<!--
[[bool-query]]
=== Combining Queries

In <<combining-filters>> we discussed how to((("full text search", "combining queries"))), use the `bool` filter to combine
multiple filter clauses with `and`, `or`, and `not` logic.  In query land, the
`bool` query does a similar job but with one important difference.

Filters make a binary decision: should this document be included in the
results list or not? Queries, however, are more subtle. They decide not only
whether to include a document, but also how _relevant_ that document is.

Like the filter equivalent, the `bool` query accepts((("bool query"))) multiple query clauses
under the `must`, `must_not`, and `should` parameters.  For instance:

[source,js]
--------------------------------------------------
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "quick" }},
      "must_not": { "match": { "title": "lazy"  }},
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ]
    }
  }
}
--------------------------------------------------
// SENSE: 100_Full_Text_Search/15_Bool_query.json

The results from the preceding query include any document whose `title` field
contains the term `quick`, except for those that also contain `lazy`. So
far, this is pretty similar to how the `bool` filter works.

The difference comes in with the two `should` clauses, which say that: a document
is _not required_ to contain ((("should clause", "in bool queries")))either `brown` or `dog`, but if it does, then
it should be considered _more relevant_:

[source,js]
--------------------------------------------------
{
  "hits": [
     {
        "_id":      "3",
        "_score":   0.70134366, <1>
        "_source": {
           "title": "The quick brown fox jumps over the quick dog"
        }
     },
     {
        "_id":      "1",
        "_score":   0.3312608,
        "_source": {
           "title": "The quick brown fox"
        }
     }
  ]
}
--------------------------------------------------

<1> Document 3 scores higher because it contains both `brown` and `dog`.

==== Score Calculation

The `bool` query calculates((("relevance scores", "calculation in bool queries")))((("bool query", "score calculation"))) the relevance `_score` for each document by adding
together the `_score` from all of the matching `must` and `should` clauses,
and then dividing by the total number of `must` and `should` clauses.

The `must_not` clauses do not affect ((("must_not clause", "in bool queries")))the score; their only purpose is to
exclude documents that might otherwise have been included.

==== Controlling Precision

All the `must` clauses must match, and all the `must_not` clauses must not
match, but how many `should` clauses((("bool query", "controlling precision")))((("full text search", "combining queries", "controlling precision")))((("precision", "controlling for bool query"))) should match? By default, none of the `should` clauses are required to match, with one
exception: if there are no `must` clauses, then at least one `should` clause
must match.

Just as we can control the <<match-precision,precision of the `match` query>>,
we can control how many `should` clauses need to match by using the
`minimum_should_match` parameter,((("minimum_should_match parameter", "in bool queries"))) either as an absolute number or as a
percentage:

[source,js]
--------------------------------------------------
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "brown" }},
        { "match": { "title": "fox"   }},
        { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 <1>
    }
  }
}
--------------------------------------------------
// SENSE: 100_Full_Text_Search/15_Bool_query.json

<1> This could also be expressed as a percentage.

The results would include only documents whose `title` field contains `"brown"
AND "fox"`, `"brown" AND "dog"`, or `"fox" AND "dog"`. If a document contains
all three, it would be considered more relevant than those that contain
just two of the three.
-->

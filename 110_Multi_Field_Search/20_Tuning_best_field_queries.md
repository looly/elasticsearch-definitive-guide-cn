### 最佳字段查询的调优

如果用户((("multifield search", "best fields queries", "tuning")))((("best fields queries", "tuning")))搜索的是"quick pets"，那么会发生什么呢？两份文档都包含了单词quick，但是只有文档2包含了单词pets。两份文档都没能在一个字段中同时包含搜索的两个单词。

一个像下面那样的简单dis_max查询会选择出拥有最佳匹配字段的查询子句，而忽略其他的查询子句：
```Javascript
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ]
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

```Javascript
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.12713557, <1>
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     },
     {
        "_id": "2",
        "_score": 0.12713557, <1>
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     }
   ]
}
```
<1> 可以发现，两份文档的分值是一模一样的。

我们期望的是同时匹配了title字段和body字段的文档能够拥有更高的排名，但是结果并非如此。需要记住：dis_max查询只是简单的使用最佳匹配查询子句得到的_score。

#### tie_breaker

但是，将其它匹配的查询子句考虑进来也是可能的。通过指定tie_breaker参数：

```Javascript
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

它会返回以下结果：

```Javascript
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.14757764, <1>
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id": "1",
        "_score": 0.124275915, <1>
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     }
   ]
}
```
<1> 现在文档2的分值比文档1稍高一些。

tie_breaker参数会让dis_max查询的行为更像是dis_max和bool的一种折中。它会通过下面的方式改变分值计算过程：

* 1.取得最佳匹配查询子句的_score。
* 2.将其它每个匹配的子句的分值乘以tie_breaker。
* 3.将以上得到的分值进行累加并规范化。

通过tie_breaker参数，所有匹配的子句都会起作用，只不过最佳匹配子句的作用更大。

> 提示：tie_breaker的取值范围是0到1之间的浮点数，取0时即为仅使用最佳匹配子句(译注：和不使用tie_breaker参数的dis_max查询效果相同)，取1则会将所有匹配的子句一视同仁。它的确切值需要根据你的数据和查询进行调整，但是一个合理的值会靠近0，(比如，0.1 -0.4)，来确保不会压倒dis_max查询具有的最佳匹配性质。


<!--
=== Tuning Best Fields Queries

What would happen if the user((("multifield search", "best fields queries", "tuning")))((("best fields queries", "tuning"))) had searched instead for ``quick pets''?  Both
documents contain the word `quick`, but only document 2 contains the word
`pets`. Neither document contains _both words_ in the _same field_.

A simple `dis_max` query like the following would ((("dis_max (disjunction max) query")))((("relevance scores", "calculation in dis_max queries")))choose the single best
matching field, and ignore the other:

[source,js]
--------------------------------------------------
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ]
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

[source,js]
--------------------------------------------------
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.12713557, <1>
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     },
     {
        "_id": "2",
        "_score": 0.12713557, <1>
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     }
   ]
}
--------------------------------------------------
<1> Note that the scores are exactly the same.

We would probably expect documents that match on both the `title` field and
the `body` field to rank higher than documents that match on just one field,
but this isn't the case. Remember: the `dis_max` query simply uses the
`_score` from the _single_ best-matching clause.

==== tie_breaker

It is possible, however, to((("dis_max (disjunction max) query", "using tie_breaker parameter")))((("relevance scores", "calculation in dis_max queries", "using tie_breaker parameter"))) also take the `_score` from the other matching
clauses into account, by specifying ((("tie_breaker parameter")))the `tie_breaker` parameter:

[source,js]
--------------------------------------------------
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

This gives us the following results:

[source,js]
--------------------------------------------------
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.14757764, <1>
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id": "1",
        "_score": 0.124275915, <1>
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     }
   ]
}
--------------------------------------------------
<1> Document 2 now has a small lead over document 1.

The `tie_breaker` parameter makes the `dis_max` query behave more like a
halfway house between `dis_max` and `bool`. It changes the score calculation
as follows:

1. Take the `_score` of the best-matching clause.
2. Multiply the score of each of the other matching clauses by the `tie_breaker`.
3. Add them all together and normalize.

With the `tie_breaker`, all matching clauses count, but the best-matching
clause counts most.

[NOTE]
====
The `tie_breaker` can be a floating-point value between `0` and `1`, where `0`
uses just the best-matching clause((("tie_breaker parameter", "value of"))) and `1` counts all matching clauses
equally.  The exact value can be tuned based on your data and queries, but a
reasonable value should be close to zero, (for example, `0.1 - 0.4`), in order not to
overwhelm the best-matching nature of `dis_max`.
====


 -->
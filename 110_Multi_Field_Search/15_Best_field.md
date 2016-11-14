### 最佳字段

假设我们有一个让用户搜索博客文章的网站(允许多字段搜索，最佳字段查询)，就像这两份文档一样：

```Javascript
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
```
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

用户输入了"Brown fox"，然后按下了搜索键。我们无法预先知道用户搜索的词条会出现在博文的title或者body字段中，但是用户是在搜索和他输入的单词相关的内容。右眼观察，以上的两份文档中，文档2似乎匹配的更好一些，因为它包含了用户寻找的两个单词。

让我们运行下面的bool查询：

```Javascript
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

然后我们发现文档1的分值更高：

```Javascript
{
  "hits": [
     {
        "_id":      "1",
        "_score":   0.14809652,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     },
     {
        "_id":      "2",
        "_score":   0.09256032,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     }
  ]
}
```

要理解原因，想想bool查询是如何计算得到其分值的：

* 1.运行should子句中的两个查询
* 2.相加查询返回的分值
* 3.将相加得到的分值乘以匹配的查询子句的数量
* 4.除以总的查询子句的数量

文档1在两个字段中都包含了brown，因此两个match查询都匹配成功并拥有了一个分值。文档2在body字段中包含了brown以及fox，但是在title字段中没有出现任何搜索的单词。因此对body字段查询得到的高分加上对title字段查询得到的零分，然后在乘以匹配的查询子句数量1，最后除以总的查询子句数量2，导致整体分值比文档1的低。

在这个例子中，title和body字段是互相竞争的。我们想要找到一个最佳匹配(Best-matching)的字段。

如果我们不是合并来自每个字段的分值，而是使用最佳匹配字段的分值作为整个查询的整体分值呢？这就会让包含有我们寻找的两个单词的字段有更高的权重，而不是在不同的字段中重复出现的相同单词。

#### dis_max查询

相比使用bool查询，我们可以使用dis_max查询(Disjuction Max Query)。Disjuction的意思"OR"(而Conjunction的意思是"AND")，因此Disjuction Max Query的意思就是返回匹配了任何查询的文档，并且分值是产生了最佳匹配的查询所对应的分值：

```Javascript
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

它会产生我们期望的结果:

```Javascript
{
  "hits": [
     {
        "_id":      "2",
        "_score":   0.21509302,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id":      "1",
        "_score":   0.12713557,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     }
  ]
}
```




<!-- === Best Fields

Imagine that we have a website that allows ((("multifield search", "best fields queries")))((("best fields queries")))users to search blog posts, such
as these two documents:

[source,js]
--------------------------------------------------
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

The user types in the words ``Brown fox'' and clicks Search.   We don't
know ahead of time if the user's search terms will be found in the `title` or
the `body` field of the post, but it is likely that the user is searching for
related words.  To our eyes, document 2 appears to be the better match, as it
contains both words that we are looking for.

Now we run the following `bool` query:

[source,js]
--------------------------------------------------
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

And we find that this query gives document 1 the higher score:

[source,js]
--------------------------------------------------
{
  "hits": [
     {
        "_id":      "1",
        "_score":   0.14809652,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     },
     {
        "_id":      "2",
        "_score":   0.09256032,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     }
  ]
}
--------------------------------------------------

To understand why, think about how the `bool` query ((("bool query", "relevance score calculation")))((("relevance scores", "calculation in bool queries")))calculates its score:

1. It runs both of the queries in the `should` clause.
2. It adds their scores together.
3. It multiplies the total by the number of matching clauses.
4. It divides the result by the total number of clauses (two).

Document 1 contains the word `brown` in both fields, so both `match` clauses
are successful and have a score.  Document 2 contains both `brown` and
`fox` in the `body` field but neither word in the `title` field. The high
score from the `body` query is added to the zero score from the `title` query,
and multiplied by one-half, resulting in a lower overall score than for document 1.

In this example, the `title` and `body` fields are competing with each other.
We want to find the single _best-matching_ field.

What if, instead of combining the scores from each field, we used the score
from the _best-matching_ field as the overall score for the query?  This would
give preference to a single field that contains _both_ of the words we are
looking for, rather than the same word repeated in different fields.

[[dis-max-query]]
==== dis_max Query

Instead of the `bool` query, we can use the  `dis_max` or _Disjunction Max
Query_.  Disjunction means _or_((("dis_max (disjunction max) query"))) (while conjunction means _and_) so the
Disjunction Max Query simply means _return documents that match any of these
queries, and return the score of the best matching query_:

[source,js]
--------------------------------------------------
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/15_Best_fields.json

This produces the results that we want:

[source,js]
--------------------------------------------------
{
  "hits": [
     {
        "_id":      "2",
        "_score":   0.21509302,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id":      "1",
        "_score":   0.12713557,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     }
  ]
}
--------------------------------------------------
-->

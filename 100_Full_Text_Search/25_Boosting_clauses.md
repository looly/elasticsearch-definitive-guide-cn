<!--秀川译-->
###提高查询得分

当然，`bool`查询并不仅仅是组合多个简单的一个词的`match`查询。他可以组合任何其他查询，包括`bool`查询。`bool`查询通常会通过组合几个不同查询的得分为每个文档调整相关性得分。

假设我们想查找关于"full-text search"的文档，但是我们又想给涉及到“Elasticsearch”或者“Lucene”的文档更高的权重。我们的用意是想涉及到"Elasticsearch" 或者 "Lucene"的文档的相关性得分会比那些没有涉及到的文档的得分要高，也就是说这些文档会出现在结果集更靠前的位置。

一个简单的`bool`查询允许我们写出像下面一样的非常复杂的逻辑：
```javascript
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {
                    "content": { (1)
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [ (2)
                { "match": { "content": "Elasticsearch" }},
                { "match": { "content": "Lucene"        }}
            ]
        }
    }
}
```
1. `content`字段必须包含`full`,`text`,`search`这三个单词。
2. 如果`content`字段也包含了“Elasticsearch”或者“Lucene”，则文档会有一个更高的得分。

匹配的`should`子句越多，文档的相关性就越强。到目前为止一切都很好。但是如果我们想给包含“Lucene”一词的文档比较高的得分，甚至给包含“Elasticsearch”一词更高的得分要怎么做呢？

我们可以在任何查询子句中指定一个`boost`值来控制相对权重，默认值为1。一个大于1的`boost`值可以提高查询子句的相对权重。因此我们可以像下面一样重写之前的查询：
```javascript
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  (1)
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 (2)
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 (3)
                    }
                }}
            ]
        }
    }
}
```
1. 这些查询子句的`boost`值为默认值`1`。
2. 这个子句是最重要的，因为他有最高的`boost`值。
3. 这个子句比第一个查询子句的要重要，但是没有“Elasticsearch”子句重要。

> 注意：
> 
> 1. `boost`参数用于提高子句的相对权重（`boost`值大于`1`）或者降低子句的相对权重（`boost`值在`0`-`1`之间），但是提高和降低并非是线性的。换句话说，`boost`值为2并不能够使结果变成两部的得分。
> 
> 2. 另外，`boost`值被使用了以后新的得分是标准的。每个查询类型都会有一个独有的标准算法，算法的详细内容并不在本书的范畴。简单的概括一下，一个更大的`boost`值可以得到一个更高的得分。
> 
> 3. 如果你自己实现了没有基于TF/IDF的得分模型，但是你想得到更多的对于提高得分过程的控制，你可以使用`function_score`查询来调整一个文档的boost值而不用通过标准的步骤。

我们会在下一章介绍更多的组合查询，[【multi-field-search】](https://github.com/looly/elasticsearch-definitive-guide-cn/tree/master/110_Multi_Field_Search)。但是首先让我们一起来看一下查询的另外一个重要的特征：文本分析。
<!--
=== Boosting Query Clauses

Of course, the `bool` query isn't restricted ((("full text search", "boosting query clauses")))to combining simple one-word
`match` queries. It can combine any other query, including other `bool`
queries.((("relevance scores", "controlling weight of query clauses")))  It is commonly used to fine-tune the relevance `_score` for each
document by combining the scores from several distinct queries.

Imagine that we want to search for documents((("bool query", "boosting weight of query clauses")))((("weight", "controlling for query clauses"))) about "full-text search,"  but we
want to give more _weight_ to documents that also mention "Elasticsearch" or
"Lucene." By _more weight_, we mean that documents mentioning
"Elasticsearch" or "Lucene" will receive a higher relevance `_score` than
those that don't, which means that they will appear higher in the list of
results.

A simple `bool` _query_ allows us to write this fairly complex logic as follows:

[source,js]
--------------------------------------------------
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {
                    "content": { <1>
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [ <2>
                { "match": { "content": "Elasticsearch" }},
                { "match": { "content": "Lucene"        }}
            ]
        }
    }
}
--------------------------------------------------
// SENSE: 100_Full_Text_Search/25_Boost.json

<1> The `content` field must contain all of the words `full`, `text`, and `search`.
<2> If the `content` field also contains `Elasticsearch` or `Lucene`,
    the document will receive a higher `_score`.

The more `should` clauses that match, the more relevant the document.  So far,
so good.

But what if we want to give more weight to the docs that contain `Lucene` and
even more weight to the docs containing `Elasticsearch`?

We can control ((("boost parameter")))the relative weight of any query clause by specifying a `boost`
value, which defaults to `1`. A `boost` value greater than `1` increases the
relative weight of that clause.  So we could  rewrite the preceding query as
follows:

[source,js]
--------------------------------------------------
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  <1>
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 <2>
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 <3>
                    }
                }}
            ]
        }
    }
}
--------------------------------------------------
// SENSE: 100_Full_Text_Search/25_Boost.json

<1> These clauses use the default `boost` of `1`.
<2> This clause is the most important, as it has the highest `boost`.
<3> This clause is more important than the default, but not as important
    as the `Elasticsearch` clause.

[NOTE]
[[boost-normalization]]
====
The `boost` parameter is used to increase((("boost parameter", "score normalied after boost applied"))) the relative weight of a clause
(with a `boost` greater than `1`) or decrease the relative weight (with a
`boost` between `0` and `1`), but the increase or decrease is not linear. In
other words, a `boost` of `2` does not result in double the `_score`.

Instead, the new `_score` is _normalized_ after((("normalization", "score normalied after boost applied"))) the boost is applied. Each
type of query has its own normalization algorithm, and the details are beyond
the scope of this book. Suffice to say that a higher `boost` value results in
a higher `_score`.

If you are implementing your own scoring model not based on TF/IDF and you
need more control over the boosting process, you can use the
<<function-score-query,`function_score` query>> to((("function_score query"))) manipulate a document's
boost without the normalization step.
====

We present other ways of combining queries in the next chapter,
<<multi-field-search>>. But first, let's take a look at the other important
feature of queries: text analysis.
-->

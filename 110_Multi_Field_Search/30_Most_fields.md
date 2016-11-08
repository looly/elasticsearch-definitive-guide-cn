#### 多数字段(Most Fields)

全文搜索是一场召回率(Recall) - 返回所有相关的文档，以及准确率(Precision) - 不返回无关文档，之间的战斗。目标是在结果的第一页给用户呈现最相关的文档。

为了提高召回率，我们会广撒网 - 不仅包括精确匹配了用户搜索词条的文档，还包括了那些我们认为和查询相关的文档。如果一个用户搜索了"quick brown fox"，一份含有fast foxes的文档也可以作为一个合理的返回结果。

如果我们拥有的相关文档仅仅是含有fast foxes的文档，那么它会出现在结果列表的顶部。但是如果我们有100份含有quick brown fox的文档，那么含有fast foxes的文档的相关性就会变低，我们希望它出现在结果列表的后面。在包含了许多可能的匹配后，我们需要确保相关度高的文档出现在顶部。

一个用来调优全文搜索相关性的常用技术是将同样的文本以多种方式索引，每一种索引方式都提供了不同相关度的信号(Signal)。主要字段(Main field)中含有的词条的形式是最宽泛的(Broadest-matching)，用来尽可能多的匹配文档。比如，我们可以这样做：

* 使用一个词干提取器来将jumps，jumping和jumped索引成它们的词根：jump。然后当用户搜索的是jumped时，我们仍然能够匹配含有jumping的文档。
* 包含同义词，比如jump，leap和hop。
* 移除变音符号或者声调符号：比如，ésta，está和esta都会以esta被索引。
但是，如果我们有两份文档，其中之一含有jumped，而另一份含有jumping，那么用户会希望第一份文档的排序会靠前，因为它含有用户输入的精确值。

我们可以通过将相同的文本索引到其它字段来提供更加精确的匹配。一个字段可以包含未被提取词干的版本，另一个则是含有变音符号的原始单词，然后第三个使用了shingles，用来提供和[单词邻近度](https://www.elastic.co/guide/en/elasticsearch/guide/current/proximity-matching.html)相关的信息。这些其它字段扮演的角色就是信号(Signals)，它们用来增加每个匹配文档的相关度分值。能够匹配的字段越多，相关度就越高。

如果一份文档能够匹配具有最宽泛形式的主要字段(Main field)，那么它就会被包含到结果列表中。如果它同时也匹配了信号字段，它会得到一些额外的分值用来将它移动到结果列表的前面。

我们会在本书的后面讨论同义词，单词邻近度，部分匹配以及其他可能的信号，但是我们会使用提取了词干和未提取词干的字段的简单例子来解释这个技术。

#### 多字段映射(Multifield Mapping)

第一件事就是将我们的字段索引两次：一次是提取了词干的形式，一次是未提取词干的形式。为了实现它，我们会使用多字段(Multifields)，在字符串排序和[多字段]()中我们介绍过：

```Javascript
DELETE /my_index

PUT /my_index
{
    "settings": { "number_of_shards": 1 }, <1>
    "mappings": {
        "my_type": {
            "properties": {
                "title": { <2>
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { <3>
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

<1> See <<[关联失效(相关性被破坏](/100_Full_Text_Search/35_Relevance_is_broken.md)>>.
<2> title字段使用了english解析器进行词干提取。 
<3> title.std字段则使用的是standard解析器，因此它没有进行词干提取。

下一步，我们会索引一些文档：

```Javascript
PUT /my_index/my_type/1
{ "title": "My rabbit jumps" }

PUT /my_index/my_type/2
{ "title": "Jumping jack rabbits" }
```
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

以下是一个简单的针对title字段的match查询，它查询jumping rabbits：

```Javascript
GET /my_index/_search
{
   "query": {
        "match": {
            "title": "jumping rabbits"
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

它会变成一个针对两个提干后的词条jump和rabbit的查询，这要得益于english解析器。两份文档的title字段都包含了以上两个词条，因此两份文档的分值是相同的：

```Javascript
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.42039964,
        "_source": {
           "title": "My rabbit jumps"
        }
     },
     {
        "_id": "2",
        "_score": 0.42039964,
        "_source": {
           "title": "Jumping jack rabbits"
        }
     }
  ]
}
```

如果我们只查询title.std字段，那么只有文档2会匹配。但是，当我们查询两个字段并将它们的分值通过bool查询进行合并的话，两份文档都能够匹配(title字段也匹配了)，而文档2的分值会更高一些(匹配了title.std字段)：

```Javascript
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", <1>
            "fields": [ "title", "title.std" ]
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

<1>  在上述查询中，由于我们想合并所有匹配字段的分值，因此使用的类型为most_fields。这会让multi_match查询将针对两个字段的查询子句包含在一个bool查询中，而不是包含在一个dis_max查询中。

```Javascript
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.8226396, <1>
        "_source": {
           "title": "Jumping jack rabbits"
        }
     },
     {
        "_id": "1",
        "_score": 0.10741998, <1>
        "_source": {
           "title": "My rabbit jumps"
        }
     }
  ]
}
```
<1> 文档2的分值比文档1的高许多。

我们使用了拥有宽泛形式的title字段来匹配尽可能多的文档 - 来增加召回率(Recall)，同时也使用了title.std字段作为信号来让最相关的文档能够拥有更靠前的排序(译注：增加了准确率(Precision))。

每个字段对最终分值的贡献可以通过指定boost值进行控制。比如，我们可以提升title字段来让该字段更加重要，这也减小了其它信号字段的影响：

```Javascript
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] <1>
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

<1> boost=10让title字段的相关性比title.std更重要。


<!--
[[most-fields]]
=== Most Fields

Full-text search is a battle between _recall_&#x2014;returning all the
documents that are ((("most fields queries")))((("multifield search", "most fields queries")))relevant--and _precision_&#x2014;not returning irrelevant
documents.  The goal is to present the user with the most relevant documents
on the first page of results.

To improve recall, we cast((("recall", "improving in full text searches"))) the net wide--we include not only
documents that match the user's search terms exactly, but also
documents that we believe to be pertinent to the query.  If a user searches
for ``quick brown fox,'' a document that contains `fast foxes` may well be
a reasonable result to return.

If the only pertinent document that we have is the one containing `fast
foxes`, it will appear at the top of the results list.  But of course, if
we have 100 documents that contain the words `quick brown fox`, then the
`fast foxes` document may be considered less relevant, and we would want to
push it further down the list.  After including many potential matches, we
need to ensure that the best ones rise to the top.

A common technique for fine-tuning full-text relevance((("relevance", "fine-tuning full text relevance"))) is to index the same
text in multiple ways, each of which provides a different relevance _signal_. The main field would contain terms in their broadest-matching form to match as
many documents as possible.  For instance, we could do the following:

*   Use a stemmer to index `jumps`, `jumping`, and `jumped` as their root
    form: `jump`.  Then it doesn't matter if the user searches for
    `jumped`; we could still match documents containing `jumping`.

*   Include synonyms like `jump`, `leap`, and `hop`.

*   Remove diacritics, or accents: for example, `ésta`, `está`, and `esta` would
    all be indexed without accents as `esta`.

However, if we have two documents, one of which contains `jumped` and the
other `jumping`, the user would probably expect the first document to rank
higher, as it contains exactly what was typed in.

We can achieve this by indexing the same text in other fields to provide more-precise matching.  One field may contain the unstemmed version, another the
original word with diacritics, and a third might use _shingles_ to provide
information about <<proximity-matching,word proximity>>. These other fields
act as _signals_ that increase the relevance score of each matching document.
The more fields that match, the better.

A document is included in the results list if it matches the broad-matching
main field. If it also matches the _signal_ fields, it gets extra
points and is pushed up the results list.

We discuss synonyms, word proximity, partial-matching and other potential
signals later in the book, but we will use the simple example of stemmed and
unstemmed fields to illustrate this technique.

==== Multifield Mapping

The first thing to do is to set up our ((("most fields queries", "multifield mapping")))((("mapping (types)", "multifield mapping")))field to be indexed twice: once in a
stemmed form and once in an unstemmed form.  To do this, we will use 
_multifields_, which we introduced in <<multi-fields>>:


[source,js]
--------------------------------------------------
DELETE /my_index

PUT /my_index
{
    "settings": { "number_of_shards": 1 }, <1>
    "mappings": {
        "my_type": {
            "properties": {
                "title": { <2>
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { <3>
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

<1> See <<relevance-is-broken>>.
<2> The `title` field is stemmed by the `english` analyzer.
<3> The `title.std` field uses the `standard` analyzer and so is not stemmed.

Next we index some documents:

[source,js]
--------------------------------------------------
PUT /my_index/my_type/1
{ "title": "My rabbit jumps" }

PUT /my_index/my_type/2
{ "title": "Jumping jack rabbits" }
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

Here is a simple `match` query on the `title` field for `jumping rabbits`:

[source,js]
--------------------------------------------------
GET /my_index/_search
{
   "query": {
        "match": {
            "title": "jumping rabbits"
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

This becomes a query for the two stemmed terms `jump` and `rabbit`, thanks to the
`english` analyzer. The `title` field of both documents contains both of those
terms, so both documents receive the same score:

[source,js]
--------------------------------------------------
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.42039964,
        "_source": {
           "title": "My rabbit jumps"
        }
     },
     {
        "_id": "2",
        "_score": 0.42039964,
        "_source": {
           "title": "Jumping jack rabbits"
        }
     }
  ]
}
--------------------------------------------------

If we were to query just the `title.std` field, then only document 2 would
match.  However, if we were to query both fields and to _combine_ their scores
by using the `bool` query, then both documents would match (thanks to the `title`
field) and document 2 would score higher (thanks to the `title.std` field):

[source,js]
--------------------------------------------------
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", <1>
            "fields": [ "title", "title.std" ]
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

<1>  We want to combine the scores from all matching fields, so we use the
     `most_fields` type.  This causes the `multi_match` query to wrap the two
     field-clauses in a `bool` query instead of a `dis_max` query.

[source,js]
--------------------------------------------------
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.8226396, <1>
        "_source": {
           "title": "Jumping jack rabbits"
        }
     },
     {
        "_id": "1",
        "_score": 0.10741998, <1>
        "_source": {
           "title": "My rabbit jumps"
        }
     }
  ]
}
--------------------------------------------------
<1> Document 2 now scores much higher than document 1.

We are using the broad-matching `title` field to include as many documents as
possible--to increase recall--but we use the `title.std` field as a
_signal_ to push the most relevant results to the top.

The contribution of each field to the final score can be controlled by
specifying custom `boost` values. For instance, we could boost the `title`
field to make it the most important field, thus reducing the effect of any
other signal fields:

[source,js]
--------------------------------------------------
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] <1>
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/30_Most_fields.json

<1> The `boost` value of `10` on the `title` field makes that field relatively
    much more important than the `title.std` field.

-->



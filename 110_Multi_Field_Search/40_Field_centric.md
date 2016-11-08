#### 以字段为中心的查询(Field-centric Queries)

上述提到的三个问题都来源于most_fields是以字段为中心(Field-centric)，而不是以词条为中心(Term-centric)：它会查询最多匹配的字段(Most matching fields)，而我们真正感兴趣的最匹配的词条(Most matching terms)。

> 提示：best_fields同样是以字段为中心的，因此它也存在相似的问题。
首先我们来看看为什么存在这些问题，以及如何解决它们。

##### 问题1：在多个字段中匹配相同的单词

考虑一下most_fields查询是如何执行的：ES会为每个字段生成一个match查询，然后将它们包含在一个bool查询中。

我们可以将查询传入到validate-query API中进行查看：

```Javascript
GET /_validate/query?explain
{
  "query": {
    "multi_match": {
      "query":   "Poland Street W1V",
      "type":    "most_fields",
      "fields":  [ "street", "city", "country", "postcode" ]
    }
  }
}
```
// SENSE: 110_Multi_Field_Search/40_Entity_search_problems.json

它会产生下面的解释(explaination)：

    (street:poland   street:street   street:w1v)
    (city:poland     city:street     city:w1v)
    (country:poland  country:street  country:w1v)
    (postcode:poland postcode:street postcode:w1v)

你可以发现能够在两个字段中匹配poland的文档会比在一个字段中匹配了poland和street的文档的分值要高。

##### 问题2：减少长尾

在[精度控制(Controlling Precision)](../100_Full_Text_Search/15_Combining_queries.md)一节中，我们讨论了如何使用and操作符和minimum_should_match参数来减少相关度低的文档数量：

```Javascript
{
    "query": {
        "multi_match": {
            "query":       "Poland Street W1V",
            "type":        "most_fields",
            "operator":    "and", <1>
            "fields":      [ "street", "city", "country", "postcode" ]
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/40_Entity_search_problems.json

<1> 所有的term必须存在。

但是，使用best_fields或者most_fields，这些参数会被传递到生成的match查询中。该查询的解释如下(译注：通过validate-query API)：

    (+street:poland   +street:street   +street:w1v)
    (+city:poland     +city:street     +city:w1v)
    (+country:poland  +country:street  +country:w1v)
    (+postcode:poland +postcode:street +postcode:w1v)

换言之，使用and操作符时，所有的单词都需要出现在相同的字段中，这显然是错的！这样做可能不会有任何匹配的文档。

##### 问题3：词条频度

在[什么是相关度(What is Relevance(relevance-intro))](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html)一节中，我们解释了默认用来计算每个词条的相关度分值的相似度算法TF/IDF：

* 词条频度(Term Frequency)::

在一份文档中，一个词条在一个字段中出现的越频繁，文档的相关度就越高。

* 倒排文档频度(Inverse Document Frequency)::

一个词条在索引的所有文档的字段中出现的越频繁，词条的相关度就越低。
当通过多字段进行搜索时，TF/IDF会产生一些令人惊讶的结果。

考虑使用first_name和last_name字段搜索"Peter Smith"的例子。Peter是一个常见的名字，Smith是一个常见的姓氏 - 它们的IDF都较低。但是如果在索引中有另外一个名为Smith Williams的人呢？Smith作为名字是非常罕见的，因此它的IDF值会很高！

像下面这样的一个简单查询会将Smith Williams放在Peter Smith前面(译注：含有Smith Williams的文档分值比含有Peter Smith的文档分值高)，尽管Peter Smith明显是更好的匹配：

```Javascript
{
    "query": {
        "multi_match": {
            "query":       "Peter Smith",
            "type":        "most_fields",
            "fields":      [ "*_name" ]
        }
    }
}
```
// SENSE: 110_Multi_Field_Search/40_Bad_frequencies.json

smith在first_name字段中的高IDF值会压倒peter在first_name字段和smith在last_name字段中的两个低IDF值。

##### 解决方案

这个问题仅在我们处理多字段时存在。如果我们将所有这些字段合并到一个字段中，该问题就不复存在了。我们可以向person文档中添加一个full_name字段来实现：

```Javascript
{
    "first_name":  "Peter",
    "last_name":   "Smith",
    "full_name":   "Peter Smith"
}
```

当我们只查询full_name字段时：

* 拥有更多匹配单词的文档会胜过那些重复出现一个单词的文档。
* minimum_should_match和operator参数能够正常工作。
* first_name和last_name的倒排文档频度会被合并，因此smith无论是first_name还是last_name都不再重要。
尽管这种方法能工作，可是我们并不想存储冗余数据。因此，ES为我们提供了两个解决方案 - 一个在索引期间，一个在搜索期间。下一节对它们进行讨论。






<!--
[[field-centric]]
=== Field-Centric Queries

All three of the preceding problems stem from  ((("field-centric queries")))((("multifield search", "field-centric queries, problems with")))((("most fields queries", "problems with field-centric queries")))`most_fields` being
_field-centric_ rather than _term-centric_: it looks for the  most matching
_fields_, when really what we're interested is the most matching _terms_.


NOTE: The `best_fields` type is also field-centric((("best fields queries", "problems with field-centric queries"))) and suffers from similar problems.


First we'll look at why these problems exist, and then how we can combat them.

==== Problem 1: Matching the Same Word in Multiple Fields

Think about how the `most_fields` query is executed: Elasticsearch generates a
separate `match` query for each field and then wraps these match queries in an outer `bool` query.

We can see this by passing our query through the `validate-query` API:

[source,js]
--------------------------------------------------
GET /_validate/query?explain
{
  "query": {
    "multi_match": {
      "query":   "Poland Street W1V",
      "type":    "most_fields",
      "fields":  [ "street", "city", "country", "postcode" ]
    }
  }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/40_Entity_search_problems.json

which yields this `explanation`:

    (street:poland   street:street   street:w1v)
    (city:poland     city:street     city:w1v)
    (country:poland  country:street  country:w1v)
    (postcode:poland postcode:street postcode:w1v)


You can see that a document matching just the word `poland` in _two_ fields
could score higher than a document matching `poland` and `street` in one
field.

==== Problem 2: Trimming the Long Tail

In <<match-precision>>, we talked about((("and operator", "most fields and best fields queries and")))((("minimum_should_match parameter", "most fields and best fields queries"))) using the `and` operator or the
`minimum_should_match` parameter to trim the long tail of almost irrelevant
results. Perhaps we could try this:

[source,js]
--------------------------------------------------
{
    "query": {
        "multi_match": {
            "query":       "Poland Street W1V",
            "type":        "most_fields",
            "operator":    "and", <1>
            "fields":      [ "street", "city", "country", "postcode" ]
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/40_Entity_search_problems.json

<1> All terms must be present.

However, with `best_fields` or `most_fields`, these parameters are passed down
to the generated `match` queries. The `explanation` for this query shows the
following:

    (+street:poland   +street:street   +street:w1v)
    (+city:poland     +city:street     +city:w1v)
    (+country:poland  +country:street  +country:w1v)
    (+postcode:poland +postcode:street +postcode:w1v)

In other words, using the `and` operator means that all words must exist _in
the same field_, which is clearly wrong! It is unlikely that any documents
would match this query.

==== Problem 3: Term Frequencies

In <<relevance-intro>>, we explained that the default similarity algorithm
used to calculate the relevance score ((("term frequency", "problems with field-centric queries")))for each term is TF/IDF:

Term frequency::

    The more often a term appears in a field in a single document, the more
    relevant the document.

Inverse document frequency::

    The more often a term appears in a field in all documents in the index,
    the less relevant is that term.

When searching against multiple fields, TF/IDF can((("Term Frequency/Inverse Document Frequency  (TF/IDF) similarity algorithm", "surprising results when searching against multiple fields"))) introduce some surprising
results.

Consider our example of searching for ``Peter Smith'' using the `first_name`
and `last_name` fields.((("inverse document frequency", "field-centric queries and")))  Peter is a common first name and Smith is a common
last name--both will have low IDFs.  But what if we have another person in
the index whose name is Smith Williams?  Smith as a first name is very
uncommon and so will have a high IDF!

A simple query like the following may well return Smith Williams above
Peter Smith in spite of the fact that the second person is a better match
than the first.

[source,js]
--------------------------------------------------
{
    "query": {
        "multi_match": {
            "query":       "Peter Smith",
            "type":        "most_fields",
            "fields":      [ "*_name" ]
        }
    }
}
--------------------------------------------------
// SENSE: 110_Multi_Field_Search/40_Bad_frequencies.json

The high IDF of `smith` in the first name field can overwhelm the two low IDFs
of `peter` as a first name and `smith` as a last name.

==== Solution

These problems only exist because we are dealing with multiple fields. If we
were to combine all of these fields into a single field, the problems would
vanish. We could achieve this by adding a `full_name` field to our `person`
document:

[source,js]
--------------------------------------------------
{
    "first_name":  "Peter",
    "last_name":   "Smith",
    "full_name":   "Peter Smith"
}
--------------------------------------------------

When querying just the `full_name` field:

* Documents with more matching words would trump documents with the same word
  repeated.

* The `minimum_should_match` and `operator` parameters would function as
  expected.

* The inverse document frequencies for first and last names would be combined
  so it wouldn't matter whether Smith were a first or last name anymore.

While this would work, we don't like having to store redundant data.  Instead,
Elasticsearch offers us two solutions--one at index time and one at search
time--which we discuss next.
-->

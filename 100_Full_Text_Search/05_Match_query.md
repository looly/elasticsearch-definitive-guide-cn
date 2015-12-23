### 匹配查询

不管你搜索什么内容，`match`查询是你首先需要接触的查询。它是一个高级查询，意味着`match`查询知道如何更好的处理全文检索和准确值检索。

这也就是说，`match`查询的一个主要用途是进行全文搜索。让我们通过一个小例子来看一下全文搜索是如何工作的。

The `match` query is the _go-to_ query--the first query that you should
reach for whenever you need to query any field.((("match query")))((("full text search", "match query"))) It is a high-level _full-text
query_, meaning that it knows how to deal with both full-text fields and exact-value fields.

That said, the main use case for the `match` query is for full-text search. So
let's take a look at how full-text search works with a simple example.

#### Index Some Data

首先，我们使用`bulk` API来创建和索引一些文档：

First, we'll create a new index and index some((("full text search", "match query", "indexing data"))) documents using the
<<bulk,`bulk` API>>:

```json
DELETE /my_index <1>

PUT /my_index
{ "settings": { "number_of_shards": 1 }} <2>

POST /my_index/my_type/_bulk
{ "index": { "_id": 1 }}
{ "title": "The quick brown fox" }
{ "index": { "_id": 2 }}
{ "title": "The quick brown fox jumps over the lazy dog" }
{ "index": { "_id": 3 }}
{ "title": "The quick brown fox jumps over the quick dog" }
{ "index": { "_id": 4 }}
{ "title": "Brown fox brown dog" }
```

// SENSE: 100_Full_Text_Search/05_Match_query.json

<1> 删除已经存在的索引(如果索引存在)
<2> 然后，`关联失效`这一节解释了为什么我们创建该索引的时候只使用一个主分片。

<1> Delete the index in case it already exists.
<2> Later, in <<relevance-is-broken>>, we explain why
    we created this index with only one primary shard.

#### 单词查询
==== A Single-Word Query

第一个例子解释了当使用`match`查询进行单词全文搜索时发生了什么：

Our first example explains what((("full text search", "match query", "single word query")))((("match query", "single word query"))) happens when we use the `match` query to
search within a full-text field for a single word:

```json
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "QUICK!"
        }
    }
}
```
// SENSE: 100_Full_Text_Search/05_Match_query.json

Elasticsearch通过下面的步骤执行`match`查询：

Elasticsearch executes the preceding `match` query((("analysis", "in single term match query"))) as follows:



1. _Check the field type_.
+
The `title` field is a full-text (`analyzed`) `string` field, which means that
the query string should be analyzed too.

2. _Analyze the query string_.
+
The query string `QUICK!` is passed through the standard analyzer, which
results in the single term `quick`. Because we have a just a single term,
the `match` query can be executed as a single low-level `term` query.

3. _Find matching docs_.
+
The `term` query looks up `quick` in the inverted index and retrieves the
list of documents that contain that term--in this case, documents 1, 2, and
3.

4. _Score each doc_.
+
The `term` query calculates the relevance `_score` for each matching document,
by combining the((("relevance scores", "calculating for single term match query results"))) term frequency (how often `quick` appears in the `title`
field of each document), with the inverse document frequency (how often
`quick` appears in the `title` field in _all_ documents in the index), and the
length of each field (shorter fields are considered more relevant).
See <<relevance-intro>>.

This process gives us the following (abbreviated) results:

[source,js]
--------------------------------------------------
"hits": [
 {
    "_id":      "1",
    "_score":   0.5, <1>
    "_source": {
       "title": "The quick brown fox"
    }
 },
 {
    "_id":      "3",
    "_score":   0.44194174, <2>
    "_source": {
       "title": "The quick brown fox jumps over the quick dog"
    }
 },
 {
    "_id":      "2",
    "_score":   0.3125, <2>
    "_source": {
       "title": "The quick brown fox jumps over the lazy dog"
    }
 }
]
--------------------------------------------------
<1> Document 1 is most relevant because its `title` field is short, which means
    that `quick` represents a large portion of its content.
<2> Document 3 is more relevant than document 2 because `quick` appears twice.

### 匹配查询

不管你搜索什么内容，`match`查询是你首先需要接触的查询。它是一个高级查询，意味着`match`查询知道如何更好的处理全文检索和准确值检索。

这也就是说，`match`查询的一个主要用途是进行全文搜索。让我们通过一个小例子来看一下全文搜索是如何工作的。

#### 索引一些数据

首先，我们使用`bulk` API来创建和索引一些文档：

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

1. _检查field类型_  
`title`字段是一个字符串(`analyzed`)，所以该查询字符串也需要被分析(`analyzed`)
2. 分析查询字符串  
查询词`QUICK!`经过标准分析器的分析后变成单词`quick`。因为我们只有一个查询词，因此`match`查询可以以一种低级别`term`查询的方式执行。
3. 找到匹配的文档  
`term`查询在倒排索引中搜索`quick`，并且返回包含该词的文档。在这个例子中，返回的文档是1，2，3
4. 为每个文档打分  
`term`查询综合考虑词频（每篇文档`title`字段包含`quick`的次数）、逆文档频率（在全部文档中`title`字段包含`quick`的次数）、包含`quick`的字段长度（长度越短越相关）来计算每篇文档的相关性得分`_score`。

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
这个过程之后我们将得到以下结果（简化后）：

```json
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
```

<1> 文档1最相关，因为 `title` 最短，意味着`quick`在语义中起比较大的作用。
<2> 文档3比文档2更相关，因为在文档3中`quick`出现了两次。

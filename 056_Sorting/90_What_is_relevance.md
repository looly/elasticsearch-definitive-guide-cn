## 相关性简介

我们曾经讲过，默认情况下，返回结果是按相关性倒序排列的。
但是什么是相关性？ 相关性如何计算？

每个文档都有相关性评分，用一个相对的浮点数字段 `_score` 来表示 -- `_score` 的评分越高，相关性越高。

查询语句会为每个文档添加一个 `_score` 字段。评分的计算方式取决于不同的查询类型 --
不同的查询语句用于不同的目的：`fuzzy` 查询会计算与关键词的拼写相似程度，`terms`查询会计算
找到的内容与关键词组成部分匹配的百分比，但是一般意义上我们说的全文本搜索是指计算内容与关键词的类似程度。

ElasticSearch的相似度算法被定义为 TF/IDF，即检索词频率/反向文档频率，包括一下内容：

检索词频率::

  检索词在该字段出现的频率？出现频率越高，相关性也越高。
  字段中出现过5次要比只出现过1次的相关性高。

反向文档频率::

  每个检索词在索引中出现的频率？频率越高，相关性越低。
  检索词出现在多数文档中会比出现在少数文档中的权重更低，
  即检验一个检索词在文档中的普遍重要性。

字段长度准则::

  字段的长度是多少？长度越长，相关性越低。
  检索词出现在一个短的 `title` 要比同样的词出现在一个长的 `content` 字段。

单个查询可以使用TF/IDF评分标准或其他方式，比如短语查询中检索词的距离或模糊查询里的检索词相似度。

相关性并不只是全文本检索的专利。也适用于`yes|no`的子句，匹配的子句越多，相关性评分越高。

如果多条查询子句被合并为一条复合查询语句，比如 `bool` 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。

## 理解评分标准

当调试一条复杂的查询语句时，想要理解相关性评分 `_score` 是比较困难的。ElasticSearch 在
每个查询语句中都有一个explain参数，将 `explain` 设为 `true` 就可以得到更详细的信息。

```Javascript
GET /_search?explain <1>
{
   "query"   : { "match" : { "tweet" : "honeymoon" }}
}
```
<1> `explain` 参数可以让返回结果添加一个 `_score` 评分的得来依据。

****
增加一个 `explain` 参数会为每个匹配到的文档产生一大堆额外内容，但是花时间去理解它是很有意义的。
如果现在看不明白也没关系 -- 等你需要的时候再来回顾这一节就行。下面我们来一点点的了解这块知识点。
****

首先，我们看一下普通查询返回的元数据：

```Javascript
{
    "_index" :      "us",
    "_type" :       "tweet",
    "_id" :         "12",
    "_score" :      0.076713204,
    "_source" :     { ... trimmed ... },
}
```

这里加入了该文档来自于哪个节点哪个分片上的信息，这对我们是比较有帮助的，因为词频率和
文档频率是在每个分片中计算出来的，而不是每个索引中：

```Javascript
    "_shard" :      1,
    "_node" :       "mzIVYCsqSWCG_M_ZffSs9Q",
```

然后返回值中的 `_explanation` 会包含在每一个入口，告诉你采用了哪种计算方式，并让你知道计算的结果以及其他详情：

```Javascript
"_explanation": { <1>
   "description": "weight(tweet:honeymoon in 0)
                  [PerFieldSimilarity], result of:",
   "value":       0.076713204,
   "details": [
      {
         "description": "fieldWeight in 0, product of:",
         "value":       0.076713204,
         "details": [
            {  <2>
               "description": "tf(freq=1.0), with freq of:",
               "value":       1,
               "details": [
                  {
                     "description": "termFreq=1.0",
                     "value":       1
                  }
               ]
            },
            { <3>
               "description": "idf(docFreq=1, maxDocs=1)",
               "value":       0.30685282
            },
            { <4>
               "description": "fieldNorm(doc=0)",
               "value":        0.25,
            }
         ]
      }
   ]
}
```
<1> `honeymoon` 相关性评分计算的总结
<2> 检索词频率
<3> 反向文档频率
<4> 字段长度准则

>**重要**：
> 输出 `explain` 结果代价是十分昂贵的，它只能用作调试工具
> --千万不要用于生产环境。

第一部分是关于计算的总结。告诉了我们 `"honeymoon"` 在 `tweet`字段中的检索词频率/反向文档频率或 TF/IDF，
（这里的文档 `0` 是一个内部的ID，跟我们没有关系，可以忽略。）

然后解释了计算的权重是如何计算出来的：

检索词频率:

    检索词 `honeymoon` 在 `tweet` 字段中的出现次数。

反向文档频率:

    检索词 `honeymoon` 在 `tweet` 字段在当前文档出现次数与索引中其他文档的出现总数的比率。

字段长度准则:

    文档中 `tweet` 字段内容的长度 -- 内容越长，值越小。

复杂的查询语句解释也非常复杂，但是包含的内容与上面例子大致相同。
通过这段描述我们可以了解搜索结果是如何产生的。

>**提示**：
>JSON形式的explain描述是难以阅读的
>但是转成 YAML 会好很多，只需要在参数中加上 `format=yaml`


## Explain Api

#### 文档是如何被匹配到的

当`explain`选项加到某一文档上时，它会告诉你为何这个文档会被匹配，以及一个文档为何没有被匹配。

请求路径为 `/index/type/id/_explain`, 如下所示：

```Javascript
GET /us/tweet/12/_explain
{
   "query" : {
      "filtered" : {
         "filter" : { "term" :  { "user_id" : 2           }},
         "query" :  { "match" : { "tweet" :   "honeymoon" }}
      }
   }
}
```

除了上面我们看到的完整描述外，我们还可以看到这样的描述：


```Javascript
"failure to match filter: cache(user_id:[2 TO 2])"
```

也就是说我们的 `user_id` 过滤子句使该文档不能匹配到。

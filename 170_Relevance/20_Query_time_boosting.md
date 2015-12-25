[[查询时提升]]
=== 查询时提升

在 <<prioritising-clauses,Prioritizing Clauses>>中, 我们解释了 ((("relevance", "controlling", "query time boosting")))((("boosting", "query-time")))你可以怎样在查询时使用 `boost`
来使得一个查询项比其它的更重要.
例如:

[source,json]
------------------------------
GET /_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": {
              "query": "quick brown fox",
              "boost": 2 <1>
            }
          }
        },
        {
          "match": { <2>
            "content": "quick brown fox"
          }
        }
      ]
    }
  }
}
------------------------------
<1> 查询项 `title` 的重要性是查询项 `content` 的2倍, 因为它被因数 `2` 提升了.
<2> 没有 `boost` 值的查询项会拥有一个默认因数为 `1` 的提升.

_查询时提升_ 是用于调节相关性的主要工具. 任何类型的查询都接受 `boost` 参数.
((("boost parameter", "setting value")))  把 `boost` 设置为 `2` 并不会简单的加倍最后的 `_score`; 
实际使用的 boost 值取决于标准化和一些内置的优化.  然而, 它确实意味着 boost 值为 `2` 的项的重要性是 boost 值为 `1`的项的2倍.

事实上, 对于一个实际的查询项，没有简单的算法来决定 ``正确'' 的 boost 值,它是边做边看的事.
要记得 `boost` 仅仅是影响相关性分数的因素之一; 它必须与其它因素竞争.  例如, 在之前的例子里, 
 `title` 字段相较于  `content 字段，可能已经有了一个 ``自然的'' 提升,  这归功于 ((("field-length norm"))) <<field-norm,field-length norm>> 
(title通常比相关的 content 要短), 所以，不要仅仅因为你觉得它应该被提升就盲目的提升一个字段
应用一个提升并且检查结果. 改变提升并且重新检查.

==== 提升一个索引

当在多个索引间搜索时, ((("boosting", "query-time", "boosting an index")))((("indices", "boosting an index"))) 你可以通过 `indices_boost` 参数提升这些索引中的某一个索引.
((("indices_boost parameter")))  下面的例子中使用了这种方法, 使得最近的索引文档拥有更高的权重:

[source,json]
------------------------------
GET /docs_2014_*/_search <1>
{
  "indices_boost": { <2>
    "docs_2014_10": 3,
    "docs_2014_09": 2
  },
  "query": {
    "match": {
      "text": "quick brown fox"
    }
  }
}
------------------------------
<1> 该多索引搜索包含了所有以 `docs_2014_` 开头的索引.
<2> 索引 `docs_2014_10` 中的文档将被因数 `3` 提升, `docs_2014_09` 中的文档被因数 `2` 提升, 其它匹配的索引将被默认的因数 `1` 提升.

==== t.getBoost()

boost 值可以通过 <<practical-scoring-function>> `t.getBoost()` 获得.
((("practical scoring function", "t.getBoost() method")))((("boosting", "query-time", "t.getBoost()")))((("t.getBoost() method"))) 
提升不会被应用在出现查询 DSL 的地方.  而是任何 boost 值都会被合并、传递到单独的 terms 中.
`t.getBoost()` 方法会返回任意应用到 term本身或更高阶查询链的 `boost` 值.

[TIP]
==================================================

事实上, 阅读 <<explain,`explain`>> 输出略为复杂. 你不会在`explanation`看到它提到过 `boost` 值或 `t.getBoost()`.
提升是被放入了应用于特殊term的<<query-norm,`queryNorm`>>中. 
尽管我们说 `queryNorm` 对于每一个 term 都是相同的, 但是你会发现已提升的term的 `queryNorm`
要比未提升的term 的 `queryNorm` 要高.

==================================================

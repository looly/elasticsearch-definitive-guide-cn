## 嵌套-集合
### 嵌套-集合

如同我们在查询时需要使用`nested`查询来存取嵌套对象，专门的`nested`集合使我们可以取得嵌套对象中栏位的集合：

```json
GET /my_index/blogpost/_search?search_type=count
{
  "aggs": {
    "comments": { <1>
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "by_month": {
          "date_histogram": { <2>
            "field":    "comments.date",
            "interval": "month",
            "format":   "yyyy-MM"
          },
          "aggs": {
            "avg_stars": {
              "avg": { <3>
                "field": "comments.stars"
              }
            }
          }
        }
      }
    }
  }
}
```
<1> `nested`集合`深入`嵌套对象的`comments`栏位
<2> 评论基於`comments.date`栏位被分至各个月份分段
<3> 每个月份分段单独计算星号的平均数

结果显示集合发生於嵌套文档层级：

```json
...
"aggregations": {
  "comments": {
     "doc_count": 4, <1>
     "by_month": {
        "buckets": [
           {
              "key_as_string": "2014-09",
              "key": 1409529600000,
              "doc_count": 1, <1>
              "avg_stars": {
                 "value": 4
              }
           },
           {
              "key_as_string": "2014-10",
              "key": 1412121600000,
              "doc_count": 3, <1>
              "avg_stars": {
                 "value": 2.6666666666666665
              }
           }
        ]
     }
  }
}
...
```
<1> 此处总共有四个`comments`: 一个在九月以及三个在十月

## 反向-嵌套-集合
### 反向嵌套-集合

一个`nested`集合只能存取嵌套文档中的栏位，而无法看见根文档或其他嵌套文档中的栏位。
然而，我们可以_跳出_嵌套区块，藉由`reverse_nested`集合回到父阶层。

举例来说，我们可以发现使用评论者的年龄为其加上`tags`很有趣。
`comment.age`是在嵌套栏位中，但是`tags`位於根文档：

```json
GET /my_index/blogpost/_search?search_type=count
{
  "aggs": {
    "comments": {
      "nested": { <1>
        "path": "comments"
      },
      "aggs": {
        "age_group": {
          "histogram": { <2>
            "field":    "comments.age",
            "interval": 10
          },
          "aggs": {
            "blogposts": {
              "reverse_nested": {}, <3>
              "aggs": {
                "tags": {
                  "terms": { <4>
                    "field": "tags"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```
<1> `nested`集合深入`comments`对象
<2> `histogram`集合以`comments.age`栏位聚集成每十年一个的分段
<3> `reverse_nested`集合跳回到根文档
<4> `terms`集合计算每个年龄分段的火红词语

简略的结果显示如下：

```json
..
"aggregations": {
  "comments": {
     "doc_count": 4, <1>
     "age_group": {
        "buckets": [
           {
              "key": 20, <2>
              "doc_count": 2, <2>
              "blogposts": {
                 "doc_count": 2, <3>
                 "tags": {
                    "doc_count_error_upper_bound": 0,
                    "buckets": [ <4>
                       { "key": "shares",   "doc_count": 2 },
                       { "key": "cash",     "doc_count": 1 },
                       { "key": "equities", "doc_count": 1 }
                    ]
                 }
              }
           },
...
```
<1> 共有四个评论
<2> 有两个评论的发表者年龄介於20至30之间
<3> 两个blog文章与这些评论相关
<4> 这些blog文章的火红标签是`shares`丶`cash`丶`equities`

### 什麽时候要使用嵌套对象

嵌套对象对於当有一个主要实体(如`blogpost`)，加上有限数量的紧密相关实体(如`comments`)是非常有用的。
有办法能以评论内容找到blog文章很有用，且`nested`查询及过滤器提供短查询时间连接(fast query-time joins)。

嵌套模型的缺点如下：

* 如欲新增丶修改或删除一个嵌套文档，则必须重新索引整个文档。因此越多嵌套文档造成越多的成本。

* 搜寻请求回传整个文档，而非只有匹配的嵌套文档。 虽然有个进行中的计画要支持只回传根文档及最匹配的嵌套文档，但目前并未支持。

有时你需要完整分离主要文档及其关连实体。 _父-子关系_提供这一个功能。


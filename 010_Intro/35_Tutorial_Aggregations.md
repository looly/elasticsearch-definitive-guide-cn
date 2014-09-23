## 分析
最后，我们还有一个需求需要完成：允许管理者在职员目录中分析。
Elasticsearch把这项功能叫做**聚合(aggregations)**，它允许你在数据基础上生成复杂的统计。它很像SQL中的`GROUP BY`但是功能更强大。

举个例子，让我们找到最受职员欢迎的兴趣：

```Javascript
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

忽略语法只看结果：

```Javascript
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

我们可以看到两个职员对音乐有兴趣，一个喜欢森林，一个喜欢运动。这些聚合的数据并没有被预先计算好，它们从匹配查询语句的文档中动态生成。如果我们想知道姓**"Smith"**的人什么兴趣最受欢迎，我们只需要增加合数的语句既可：

```Javascript
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```

`all_interests`已经变成只包含匹配语句的文档了：

```Javascript
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
```

聚合也允许分级汇总。例如，让我们统计每种兴趣下职员的平均年龄：

```Javascript
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

虽然这次返回的聚合结果更加复杂，但是依旧容易理解：

```Javascript
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```

输出基本上是我们之前运行聚合的一个丰富化版本。我们依旧有兴趣以及它们数量的列表，但是现在每个兴趣额外拥有`avg_age`用来显示拥有此兴趣职员的平均年龄。

即使你依旧不能理解语法，但是可以很轻松的看到如此复杂的聚合和分组能够使用这些特性完成。处理数据的能力取决于你能提取什么样的数据！

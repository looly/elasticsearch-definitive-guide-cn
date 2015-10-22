## 嵌套排序
### 以嵌套栏位排序

我们可以依照嵌套栏位中的值来排序，甚至藉由分离嵌套文档中的值。为了使其结果更加有趣，我们加入另一个记录：

```json
PUT /my_index/blogpost/2
{
  "title": "Investment secrets",
  "body":  "What they don't tell you ...",
  "tags":  [ "shares", "equities" ],
  "comments": [
    {
      "name":    "Mary Brown",
      "comment": "Lies, lies, lies",
      "age":     42,
      "stars":   1,
      "date":    "2014-10-18"
    },
    {
      "name":    "John Smith",
      "comment": "You're making it up!",
      "age":     28,
      "stars":   2,
      "date":    "2014-10-16"
    }
  ]
}
```

想像我们要取回在十月中有收到回应的blog文章，并依照所取回的各个blog文章中最少`stars`数量的顺序作排序。
这个搜寻请求如下：

```json
GET /_search
{
  "query": {
    "nested": { <1>
      "path": "comments",
      "filter": {
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  },
  "sort": {
    "comments.stars": { <2>
      "order": "asc",   <2>
      "mode":  "min",   <2>
      "nested_filter": { <3>
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  }
}
```
<1> `nested`查询限制了结果为十月份收到回应的blog文章。
<2> 结果在所有匹配的回应中依照`comment.stars`栏位的最小值(`min`)作递增(`asc`)的排序。
<3> 排序条件中的`nested_filter`与主查询`query`条件中的`nested`查询相同。 於下一个下方解释。

为什么我们要在`nested_filter`重复写上查询条件？ 原因是排序在於执行查询后才发生。
此查询匹配了在十月中有收到回应的blog文章，回传blog文章文档作为结果。
如果我们不加上`nested_filter`条件，我们最後会依照任何blog文章曾经收到过的回应作排序，而不是在十月份收到的。

### 查询多个准确值

`term` 过滤器在查询单个值时很好用，但是你可能经常需要搜索多个值。比如你想寻找 20 或 30 元的文档，该怎么做呢？

比起使用多个 `term` 过滤器，你可以用一个 `terms` 过滤器。`terms` 过滤器是 `term` 过滤器的复数版本。

它用起来和 `term` 差不多，我们现在来指定一组数值，而不是单一价格：

```json
{
    "terms" : {
        "price" : [20, 30]
    }
}
```

像 `term` 过滤器一样，我们将它放在 `filtered` 查询中：

```json
GET /my_store/products/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "terms" : { <1>
                    "price" : [20, 30]
                }
            }
        }
    }
}
```

<!-- SENSE: 080_Structured_Search/15_Terms_filter.json -->

<1> 这是前面提到的 `terms` 过滤器，放置在 `filtered` 查询中

这条查询将返回第二，第三和第四个文档：

```json
"hits" : [
    {
        "_id" :    "2",
        "_score" : 1.0,
        "_source" : {
          "price" :     20,
          "productID" : "KDKE-B-9947-#kL5"
        }
    },
    {
        "_id" :    "3",
        "_score" : 1.0,
        "_source" : {
          "price" :     30,
          "productID" : "JODL-X-1937-#pV7"
        }
    },
    {
        "_id":     "4",
        "_score":  1.0,
        "_source": {
           "price":     30,
           "productID": "QQPX-R-3956-#aD8"
        }
     }
]
```

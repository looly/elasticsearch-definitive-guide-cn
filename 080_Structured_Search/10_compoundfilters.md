### 组合过滤

前面的两个例子展示了单个过滤器的使用。现实中，你可能需要过滤多个值或字段，例如，想在 Elasticsearch 中表达这句 SQL 吗？

```sql
SELECT product
FROM   products
WHERE  (price = 20 OR productID = "XHDK-A-1293-#fJ3")
  AND  (price != 30)
```

这些情况下，你需要 `bool` 过滤器。这是以其他过滤器作为参数的_组合过滤器_，将它们结合成多种布尔组合。

#### 布尔过滤器

`bool` 过滤器由三部分组成：

```json
{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
   }
}
```

`must`：所有分句都_必须_匹配，与 `AND` 相同。

`must_not`：所有分句都_必须不_匹配，与 `NOT` 相同。

`should`：至少有一个分句匹配，与 `OR` 相同。

这样就行了！假如你需要多个过滤器，将他们放入 `bool` 过滤器就行。

提示：
  `bool` 过滤器的每个部分都是可选的（例如，你可以只保留一个 `must` 分句），而且每个部分可以包含一到多个过滤器

为了复制上面的 SQL 示例，我们将两个 `term` 过滤器放在 `bool` 过滤器的 `should` 分句下，然后用另一个分句来处理 `NOT` 条件：

```json
GET /my_store/products/_search
{
   "query" : {
      "filtered" : { <1>
         "filter" : {
            "bool" : {
              "should" : [
                 { "term" : {"price" : 20}}, <2>
                 { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} <2>
              ],
              "must_not" : {
                 "term" : {"price" : 30} <3>
              }
           }
         }
      }
   }
}
```

<!-- SENSE: 080_Structured_Search/10_Bool_filter.json -->

<1> 注意我们仍然需要用 `filtered` 查询来包裹所有条件。

<2> 这两个 `term` 过滤器是 `bool` 过滤器的_子节点_，因为它们被放在 `should` 分句下，所以至少他们要有一个条件符合。

<3> 如果一个产品价值 `30`，它就会被自动排除掉，因为它匹配了 `must_not` 分句。

我们的搜索结果返回了两个结果，分别满足了 `bool` 过滤器中的不同分句：

```json
"hits" : [
    {
        "_id" :     "1",
        "_score" :  1.0,
        "_source" : {
          "price" :     10,
          "productID" : "XHDK-A-1293-#fJ3" <1>
        }
    },
    {
        "_id" :     "2",
        "_score" :  1.0,
        "_source" : {
          "price" :     20, <2>
          "productID" : "KDKE-B-9947-#kL5"
        }
    }
]
```

<1> 匹配 `term` 过滤器 `productID = "XHDK-A-1293-#fJ3"`

<2> 匹配 `term` 过滤器 `price = 20`

#### 嵌套布尔过滤器

虽然 `bool` 是一个组合过滤器而且接受子过滤器，需明白它自己仍然只是一个过滤器。这意味着你可以在 `bool` 过滤器中嵌套 `bool` 过滤器，让你实现更复杂的布尔逻辑。

下面先给出 SQL 语句：

```sql
SELECT document
FROM   products
WHERE  productID      = "KDKE-B-9947-#kL5"
  OR (     productID = "JODL-X-1937-#pV7"
       AND price     = 30 )
```

我们可以将它翻译成一对嵌套的 `bool` 过滤器：

```json
GET /my_store/products/_search
{
   "query" : {
      "filtered" : {
         "filter" : {
            "bool" : {
              "should" : [
                { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, <1>
                { "bool" : { <1>
                  "must" : [
                    { "term" : {"productID" : "JODL-X-1937-#pV7"}}, <2>
                    { "term" : {"price" : 30}} <2>
                  ]
                }}
              ]
           }
         }
      }
   }
}
```

<!-- SENSE: 080_Structured_Search/10_Bool_filter.json -->

<1> 因为 `term` 和 `bool` 在第一个 `should` 分句中是平级的，至少需要匹配其中的一个过滤器。

<2> `must` 分句中有两个平级的 `term` 分句，所以他们俩都需要匹配。

结果得到两个文档，分别匹配一个 `should` 分句：

```json
"hits" : [
    {
        "_id" :     "2",
        "_score" :  1.0,
        "_source" : {
          "price" :     20,
          "productID" : "KDKE-B-9947-#kL5" <1>
        }
    },
    {
        "_id" :     "3",
        "_score" :  1.0,
        "_source" : {
          "price" :      30, <2>
          "productID" : "JODL-X-1937-#pV7" <2>
        }
    }
]
```

<1> `productID` 匹配第一个 `bool` 中的 `term` 过滤器。

<2> 这两个字段匹配嵌套的 `bool` 中的 `term` 过滤器。

这只是一个简单的例子，但是它展示了该怎样用布尔过滤器来构造复杂的逻辑条件。

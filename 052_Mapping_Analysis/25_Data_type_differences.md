## 映射及分析

当在索引中处理数据时，我们注意到一些奇怪的事。有些东西似乎被破坏了：

在索引中有12个tweets，只有一个包含日期`2014-09-15`，但是我们看看下面查询中的`total` hits。

```javascript
GET /_search?q=2014              # 12 个结果
GET /_search?q=2014-09-15        # 还是 12 个结果 !
GET /_search?q=date:2014-09-15   # 1  一个结果
GET /_search?q=date:2014         # 0  个结果 !
```

为什么全日期的查询返回所有的tweets，而针对`date`字段进行年度查询却什么都不返回？
为什么我们的结果因查询`_all`字段(译者注：默认所有字段中进行查询)或`date`字段而变得不同？

想必是因为我们的数据在`_all`字段的索引方式和在`date`字段的索引方式不同而导致。

让我们看看Elasticsearch在对`gb`索引中的`tweet`类型进行_mapping_(也称之为_模式定义_[注：此词有待重新定义(schema definition)])后是如何解读我们的文档结构：

```javascript
GET /gb/_mapping/tweet
```

返回：

```javascript
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "dateOptionalTime"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

Elasticsearch为对字段类型进行猜测，动态生成了字段和类型的映射关系。返回的信息显示了`date`字段被识别为`date`类型。`_all`因为是默认字段所以没有在此显示，不过我们知道它是`string`类型。

`date`类型的字段和`string`类型的字段的索引方式是不同的，因此导致查询结果的不同，这并不会让我们觉得惊讶。

你会期望每一种核心数据类型(strings, numbers, booleans及dates)以不同的方式进行索引，而这点也是现实：在Elasticsearch中他们是被区别对待的。

但是更大的区别在于_确切值_(exact values)(比如`string`类型)及_全文文本_(full text)之间。

这两者的区别才真的很重要 - 这是区分搜索引擎和其他数据库的根本差异。

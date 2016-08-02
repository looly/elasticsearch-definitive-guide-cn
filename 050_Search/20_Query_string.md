## 简易搜索

`search` API有两种表单：一种是“简易版”的**查询字符串(query string)**将所有参数通过查询字符串定义，另一种版本使用JSON完整的表示**请求体(request body)**，这种富搜索语言叫做结构化查询语句（DSL）

查询字符串搜索对于在命令行下运行**点对点(ad hoc)**查询特别有用。例如这个语句查询所有类型为`tweet`并在`tweet`字段中包含`elasticsearch`字符的文档：

```javascript
GET /_all/tweet/_search?q=tweet:elasticsearch
```

下一个语句查找`name`字段中包含`"john"`和`tweet`字段包含`"mary"`的结果。实际的查询只需要：

    +name:john +tweet:mary

但是**百分比编码(percent encoding)**（译者注：就是url编码）需要将查询字符串参数变得更加神秘：

```Javascript
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```

`"+"`前缀表示语句匹配条件**必须**被满足。类似的`"-"`前缀表示条件**必须不**被满足。所有条件如果没有`+`或`-`表示是可选的——匹配越多，相关的文档就越多。

### `_all`字段

返回包含`"mary"`字符的所有文档的简单搜索：

```javascript
GET /_search?q=mary
```

在前一个例子中，我们搜索`tweet`或`name`字段中包含某个字符的结果。然而，这个语句返回的结果在三个不同的字段中包含`"mary"`：

* 用户的名字是“Mary”
* “Mary”发的六个推文
* 针对“@mary”的一个推文

Elasticsearch是如何设法找到三个不同字段的结果的？

当你索引一个文档，Elasticsearch把所有字符串字段值连接起来放在一个大字符串中，它被索引为一个特殊的字段`_all`。例如，当索引这个文档：

```javascript
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```

这好比我们增加了一个叫做`_all`的额外字段值：

```javascript
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```

若没有指定字段，查询字符串搜索（即q=xxx）使用`_all`字段搜索。

> ### TIP
> `_all`字段对于开始一个新应用时是一个有用的特性。之后，如果你定义字段来代替`_all`字段，你的搜索结果将更加可控。当`_all`字段不再使用，你可以停用它，这个会在《全字段》章节阐述。

### 更复杂的语句

下一个搜索推特的语句：

  `_all` field
* `name`字段包含`"mary"`或`"john"`
* `date`晚于`2014-09-10`
* `_all`字段包含`"aggregations"`或`"geo"`

```javascript
+name:(mary john) +date:>2014-09-10 +(aggregations geo)
```

编码后的查询字符串变得不太容易阅读：

```javascript
?q=%2Bname%3A(mary+john)+%2Bdate%3A%3E2014-09-10+%2B(aggregations+geo)
```

就像你上面看到的例子，**简单(lite)**查询字符串搜索惊人的强大。它的查询语法，会在《查询字符串语法》章节阐述。参考文档允许我们简洁明快的表示复杂的查询。这对于命令行下一次性查询或者开发模式下非常有用。

然而，你可以看到简洁带来了隐晦和调试困难。而且它很脆弱——查询字符串中一个细小的语法错误，像`-`、`:`、`/`或`"`错位就会导致返回错误而不是结果。

最后，查询字符串搜索允许任意用户在索引中任何一个字段上运行潜在的慢查询语句，可能暴露私有信息甚至使你的集群瘫痪。

> ### TIP
> 因为这些原因，我们不建议直接暴露查询字符串搜索给用户，除非这些用户对于你的数据和集群可信。

取而代之的，生产环境我们一般依赖全功能的**请求体**搜索API，它能完成前面所有的事情，甚至更多。在了解它们之前，我们首先需要看看数据是如何在Elasticsearch中被索引的。

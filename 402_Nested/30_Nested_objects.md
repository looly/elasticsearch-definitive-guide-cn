## 嵌套-对象
### 嵌套对象

事实上在Elasticsearch中，创建丶删除丶修改一个文档是是原子性的，因此我们可以在一个文档中储存密切关联的实体。举例来说，我们可以在一个文档中储存一笔订单及其所有内容，或是储存一个Blog文章及其所有回应，藉由传递一个`comments`阵列：

```json
PUT /my_index/blogpost/1
{
  "title": "Nest eggs",
  "body":  "Making your money work...",
  "tags":  [ "cash", "shares" ],
  "comments": [ <1>
    {
      "name":    "John Smith",
      "comment": "Great article",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "More like this please",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}
```
<1> 如果我们依靠动态映射，`comments`栏位会被自动建立为一个`object`栏位。

因为所有内容都在同一个文档中，使搜寻时并不需要连接(join)blog文章与回应，因此搜寻表现更加优异。

问题在於以上的文档可能会如下所示的匹配一个搜寻：

```json
GET /_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "Alice" }},
        { "match": { "age":  28      }} <1>
      ]
    }
  }
}
```
<1> Alice是31岁，而不是28岁！

造成跨对象配对的原因如同我们在对象阵列中所讨论到，在于我们优美结构的JSON文档在索引中被扁平化为下方的 键-值 形式：

```json
{
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ],
  "comments.name":    [ alice, john, smith, white ],
  "comments.comment": [ article, great, like, more, please, this ],
  "comments.age":     [ 28, 31 ],
  "comments.stars":   [ 4, 5 ],
  "comments.date":    [ 2014-09-01, 2014-10-22 ]
}
```

`Alice`与`31` 以及 `John`与`2014-09-01` 之间的关联已经无法挽回的消失了。 
当`object`类型的栏位用于储存_单一_对象是非常有用的。
从搜寻的角度来看，对於排序一个对象阵列来说关联是不需要的东西。

这是_嵌套对象_被设计来解决的问题。 藉由映射`commments`栏位为`nested`类型而不是`object`类型，
每个嵌套对象会被索引为一个_隐藏分割文档_，例如：

```json
{ <1>
  "comments.name":    [ john, smith ],
  "comments.comment": [ article, great ],
  "comments.age":     [ 28 ],
  "comments.stars":   [ 4 ],
  "comments.date":    [ 2014-09-01 ]
}
{ <2>
  "comments.name":    [ alice, white ],
  "comments.comment": [ like, more, please, this ],
  "comments.age":     [ 31 ],
  "comments.stars":   [ 5 ],
  "comments.date":    [ 2014-10-22 ]
}
{ <3>
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ]
}
```
<1> 第一个`嵌套`对象
<2> 第二个`嵌套`对象
<3> 根或是父文档

藉由分别索引每个嵌套对象，对象的栏位中保持了其关联。 我们的查询可以只在同一个嵌套对象都匹配时才回应。

不仅如此，因嵌套对象都被索引了，连接嵌套对象至根文档的查询速度非常快--几乎与查询单一文档一样快。

这些额外的嵌套对象被隐藏起来，我们无法直接访问他们。 为了要新增丶修改或移除一个嵌套对象，我们必须重新索引整个文档。
要牢记搜寻要求的结果并不是只有嵌套对象，而是整个文档。



## 查询与过滤条件的合并

查询语句和过滤语句可以放在各自的上下文中。
在 ElasticSearch API 中我们会看到许多带有 `query` 或 `filter` 的语句。
这些语句既可以包含单条 query 语句，也可以包含一条 filter 子句。
换句话说，这些语句需要首先创建一个`query`或`filter`的上下文关系。

复合查询语句可以加入其他查询子句，复合过滤语句也可以加入其他过滤子句。
通常情况下，一条查询语句需要过滤语句的辅助，全文本搜索除外。

所以说，查询语句可以包含过滤子句，反之亦然。
以便于我们切换 query 或 filter 的上下文。这就要求我们在读懂需求的同时构造正确有效的语句。

## 带过滤的查询语句

#### 过滤一条查询语句

比如说我们有这样一条查询语句:

```Javascript
{
    "match": {
        "email": "business opportunity"
    }
}
```

然后我们想要让这条语句加入 `term` 过滤，在收信箱中匹配邮件：

```Javascript
{
    "term": {
        "folder": "inbox"
    }
}
```

`search` API中只能包含 `query` 语句，所以我们需要用 `filtered` 来同时包含
"query" 和 "filter" 子句：

```Javascript
{
    "filtered": {
        "query":  { "match": { "email": "business opportunity" }},
        "filter": { "term":  { "folder": "inbox" }}
    }
}
```

我们在外层再加入 `query` 的上下文关系：

```Javascript
GET /_search
{
    "query": {
        "filtered": {
            "query":  { "match": { "email": "business opportunity" }},
            "filter": { "term": { "folder": "inbox" }}
        }
    }
}
```

## 单条过滤语句

在 `query` 上下文中，如果你只需要一条过滤语句，比如在匹配全部邮件的时候，你可以
省略 `query` 子句：

```Javascript
GET /_search
{
    "query": {
        "filtered": {
            "filter":   { "term": { "folder": "inbox" }}
        }
    }
}
```

如果一条查询语句没有指定查询范围，那么它默认使用 `match_all` 查询，所以上面语句
的完整形式如下：

```Javascript
GET /_search
{
    "query": {
        "filtered": {
            "query":    { "match_all": {}},
            "filter":   { "term": { "folder": "inbox" }}
        }
    }
}
```


## 查询语句中的过滤

有时候，你需要在 filter 的上下文中使用一个 query 子句。下面的语句就是一条带有查询功能
的过滤语句， 这条语句可以过滤掉看起来像垃圾邮件的文档：


```Javascript
GET /_search
{
    "query": {
        "filtered": {
            "filter":   {
                "bool": {
                    "must":     { "term":  { "folder": "inbox" }},
                    "must_not": {
                        "query": { <1>
                            "match": { "email": "urgent business proposal" }
                        }
                    }
                }
            }
        }
    }
}
```
<1> 过滤语句中可以使用`query`查询的方式代替 `bool` 过滤子句。

>**提示**：
>我们很少用到的过滤语句中包含查询，保留这种用法只是为了语法的完整性。
>只有在过滤中用到全文本匹配的时候才会使用这种结构。


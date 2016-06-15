## 结构化查询 Query DSL

结构化查询是一种灵活的，多表现形式的查询语言。
Elasticsearch在一个简单的JSON接口中用结构化查询来展现Lucene绝大多数能力。
你应当在你的产品中采用这种方式进行查询。它使得你的查询更加灵活，精准，易于阅读并且易于debug。

使用结构化查询，你需要传递`query`参数：

```Javascript
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```

空查询 - `{}` - 在功能上等同于使用`match_all`查询子句，正如其名字一样，匹配所有的文档：

```Javascript
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

### 查询子句

一个查询子句一般使用这种结构：

```Javascript
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```

或指向一个指定的字段：

```Javascript
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

例如，你可以使用`match`查询子句用来找寻在`tweet`字段中找寻包含`elasticsearch`的成员：

```Javascript
{
    "match": {
        "tweet": "elasticsearch"
    }
}
```

完整的查询请求会是这样：

```Javascript
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

### 合并多子句

查询子句就像是搭积木一样，可以合并简单的子句为一个复杂的查询语句，比如：

* 叶子子句(_leaf clauses_)(比如`match`子句)用以在将查询字符串与一个字段(或多字段)进行比较

* 复合子句(_compound_)用以合并其他的子句。例如，`bool`子句允许你合并其他的合法子句，`must`，`must_not`或者`should`，如果可能的话：

```Javascript
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }}
    }
}
```

复合子句能合并 **任意**其他查询子句，包括其他的复合子句。
这就意味着复合子句可以相互嵌套，从而实现非常复杂的逻辑。

以下实例查询的是邮件正文中含有“business opportunity”字样的星标邮件或收件箱中正文中含有“business opportunity”字样的非垃圾邮件：
```Javascript
{
    "bool": {
        "must": { "match":      { "email": "business opportunity" }},
        "should": [
             { "match":         { "starred": true }},
             { "bool": {
                   "must":      { "folder": "inbox" }},
                   "must_not":  { "spam": true }}
             }}
        ],
        "minimum_should_match": 1
    }
}
```

不用担心这个例子的细节，我们将在后面详细解释它。
重点是复合子句可以合并多种子句为一个单一的查询，无论是叶子子句还是其他的复合子句。

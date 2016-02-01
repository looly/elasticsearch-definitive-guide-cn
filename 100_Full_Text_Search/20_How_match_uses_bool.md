### match匹配怎么当成布尔查询来使用

到现在为止，你可能已经意识到在一个布尔查询中多字段`match`查询仅仅包裹了已经生成的`term`查询。通过默认的`or`操作符，每个`term`查询都会像一个`should`子句一样被添加，只要有一个子句匹配就可以了。下面的两个查询是等价的：

```Javacript
{
    "match": { "title": "brown fox"}
}
```

```Javascript
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

通过`and`操作符，所有的`term`查询会像`must`子句一样被添加，因此所有的子句都必须匹配。下面的两个查询是等价的：

```Javascript
{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
```

```Javascript
{
  "bool": {
    "must": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

如果`minimum_should_match`参数被指定，`match`查询就直接被转换成一个`bool`查询，下面两个查询是等价的：

```Javascript
{
    "match": {
        "title": {
            "query":                "quick brown fox",
            "minimum_should_match": "75%"
        }
    }
}
```

```Javascript
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }},
      { "term": { "title": "quick" }}
    ],
    "minimum_should_match": 2 <1>
  }
}
```
<1>因为只有三个子句，所以 `minimum_should_match`参数在`match`查询中的值`75%`就下舍到了`2`。3个`should`子句中至少有两个子句匹配。

当然，我们通常写这些查询类型的时候还是使用`match`查询的，但是理解`match`查询在内部是怎么工作的可以让你在任何你需要使用的时候更加得心应手。有些情况仅仅使用一个`match`查询是不够的，比如给某些查询词更高的权重。这种情况我们会在下一节看个例子。


<!--
=== How match Uses bool

By now, you have probably realized that <<match-multi-word,multiword `match`
queries>> simply wrap((("match query", "use of bool query in multi-word searches")))((("bool query", "use by match query in multi-word searches")))((("full text search", "how match query uses bool query"))) the generated `term` queries in a `bool` query. With the
default `or` operator, each `term` query is added as a `should` clause, so
at least one clause must match. These two queries are equivalent:

[source,js]
--------------------------------------------------
{
    "match": { "title": "brown fox"}
}
--------------------------------------------------

[source,js]
--------------------------------------------------
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
--------------------------------------------------

With the `and` operator, all the `term` queries are added as `must` clauses,
so _all_ clauses must match. These two queries are equivalent:

[source,js]
--------------------------------------------------
{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
--------------------------------------------------

[source,js]
--------------------------------------------------
{
  "bool": {
    "must": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
--------------------------------------------------

And if the `minimum_should_match` parameter is((("minimum_should_match parameter", "match query using bool query"))) specified, it is passed
directly through to the `bool` query, making these two queries equivalent:

[source,js]
--------------------------------------------------
{
    "match": {
        "title": {
            "query":                "quick brown fox",
            "minimum_should_match": "75%"
        }
    }
}
--------------------------------------------------

[source,js]
--------------------------------------------------
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }},
      { "term": { "title": "quick" }}
    ],
    "minimum_should_match": 2 <1>
  }
}
--------------------------------------------------
<1> Because there are only three clauses, the `minimum_should_match`
    value of `75%` in the `match` query is rounded down to `2`.
    At least two out of the three `should`  clauses must match.


Of course, we would normally write these types of queries by using the `match`
query, but understanding how the `match` query works internally lets you take
control of the process when you need to. Some things can't be
done with a single `match` query, such as give more weight to some query terms
than to others. We will look at an example of this in the next section.

-->


### 处理 Null 值

回到我们早期的示例，在文档中有一个多值的字段 `tags`，一个文档可能包含一个或多个标签，或根本没有标签。如果一个字段没有值，它是怎么储存在倒排索引中的？

这是一个取巧的问题，因为答案是它根本没有存储。让我们从看一下前几节的倒排索引：

| Token         | DocIDs |
|---------------|--------|
|`open_source`  | `2`    |
|`search`       | `1`,`2`|

你怎么可能储存一个在数据结构不存在的字段呢？倒排索引是标记和包含它们的文档的一个简单列表。假如一个字段不存在，它就没有任何标记，也就意味着它无法被倒排索引的数据结构表达出来。

本质上来说，`null`，`[]`（空数组）和 `[null]` 是相等的。它们都不存在于倒排索引中！

显然，这个世界却没有那么简单，数据经常会缺失字段，或包含空值或空数组。为了应对这些情形，Elasticsearch 有一些工具来处理空值或缺失的字段。

#### `exists` 过滤器

工具箱中的第一个利器是 `exists` 过滤器，这个过滤器将返回任何包含这个字段的文档，让我们用标签来举例，索引一些示例文档：

```json
POST /my_index/posts/_bulk
{ "index": { "_id": "1"              }}
{ "tags" : ["search"]                }  <1>
{ "index": { "_id": "2"              }}
{ "tags" : ["search", "open_source"] }  <2>
{ "index": { "_id": "3"              }}
{ "other_field" : "some data"        }  <3>
{ "index": { "_id": "4"              }}
{ "tags" : null                      }  <4>
{ "index": { "_id": "5"              }}
{ "tags" : ["search", null]          }  <5>
```

<!-- SENSE: 080_Structured_Search/30_Exists_missing.json -->

<1> `tags` 字段有一个值
<2> `tags` 字段有两个值
<3> `tags` 字段不存在
<4> `tags` 字段被设为 `null`
<5> `tags` 字段有一个值和一个 `null`

结果我们 `tags` 字段的倒排索引看起来将是这样：

| Token        | DocIDs      |
|--------------|-------------|
|`open_source` | `2`         |
|`search`      | `1`,`2`,`5` |

我们的目标是找出所有设置了标签的文档，我们不关心这个标签是什么，只要它存在于文档中就行。在 SQL 语法中，我们可以用 `IS NOT NULL` 查询：

```sql
SELECT tags
FROM   posts
WHERE  tags IS NOT NULL
```

在 Elasticsearch 中，我们使用 `exists` 过滤器：

```json
GET /my_index/posts/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "exists" : { "field" : "tags" }
            }
        }
    }
}
```

<!-- SENSE: 080_Structured_Search/30_Exists_missing.json -->

查询返回三个文档：

```json
"hits" : [
    {
      "_id" :     "1",
      "_score" :  1.0,
      "_source" : { "tags" : ["search"] }
    },
    {
      "_id" :     "5",
      "_score" :  1.0,
      "_source" : { "tags" : ["search", null] } <1>
    },
    {
      "_id" :     "2",
      "_score" :  1.0,
      "_source" : { "tags" : ["search", "open source"] }
    }
]
```

<1> 文档 5 虽然包含了一个 `null` 值，仍被返回了。这个字段存在是因为一个有值的标签被索引了，所以 `null` 对这个过滤器没有影响

结果很容易理解，所以在 `tags` 字段中有值的文档都被返回了。只排除了文档 3 和 4。

#### `missing` 过滤器

`missing` 过滤器本质上是 `exists` 的反义词：它返回没有特定字段值的文档，像这条 SQL 一样：

```sql
SELECT tags
FROM   posts
WHERE  tags IS  NULL
```

让我们在前面的例子中用 `missing` 过滤器来取代 `exists`：

```json
GET /my_index/posts/_search
{
    "query" : {
        "filtered" : {
            "filter": {
                "missing" : { "field" : "tags" }
            }
        }
    }
}
```

<!-- SENSE: 080_Structured_Search/30_Exists_missing.json -->

如你所愿，我们得到了两个没有包含标签字段的文档：

```json
"hits" : [
    {
      "_id" :     "3",
      "_score" :  1.0,
      "_source" : { "other_field" : "some data" }
    },
    {
      "_id" :     "4",
      "_score" :  1.0,
      "_source" : { "tags" : null }
    }
]
```

什么时候 null 才表示 null

有时你需要能区分一个字段是没有值，还是被设置为 `null`。用上面见到的默认行为无法区分这一点，数据都不存在了。幸运的是，我们可以将明确的 `null` 值用我们选择的_占位符_来代替

当指定字符串，数字，布尔值或日期字段的映射时，你可以设置一个 `null_value` 来处理明确的 `null` 值。没有值的字段仍将被排除在倒排索引外。

当选定一个合适的 `null_value` 时，确保以下几点：

* 它与字段的类型匹配，你不能在 `date` 类型的字段中使用字符串 `null_value`
* 它需要能与这个字段可能包含的正常值区分开来，以避免真实值和 `null` 值混淆

#### 对象的 `exists/missing`

`exists` 和 `missing` 过滤器同样能在内联对象上工作，而不仅仅是核心类型。例如下面的文档：

```json
{
   "name" : {
      "first" : "John",
      "last" :  "Smith"
   }
}
```

你可以检查 `name.first` 和 `name.last` 的存在性，也可以检查 `name` 的。然而，在【映射】中，我们提到对象在内部被转成扁平化的键值结构，像下面所示：

```json
{
   "name.first" : "John",
   "name.last"  : "Smith"
}
```

所以我们是怎么使用 `exists` 或 `missing` 来检测 `name` 字段的呢，这个字段并没有真正存在于倒排索引中。

原因是像这样的一个过滤器

```json
{
    "exists" : { "field" : "name" }
}
```

实际是这样执行的

```json
{
    "bool": {
        "should": [
            { "exists": { "field": { "name.first" }}},
            { "exists": { "field": { "name.last"  }}}
        ]
    }
}
```

同样这意味着假如 `first` 和 `last` 都为空，那么 `name` 就是不存在的。

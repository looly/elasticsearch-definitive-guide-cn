### 范围

我们到现在只搜索过准确的数字，现实中，通过范围来过滤更为有用。例如，你可能希望找到所有价格高于 20 元而低于 40 元的产品。

在 SQL 语法中，范围可以如下表示：

```sql
SELECT document
FROM   products
WHERE  price BETWEEN 20 AND 40
```

Elasticsearch 有一个 `range` 过滤器，让你可以根据范围过滤：

```json
"range" : {
    "price" : {
        "gt" : 20,
        "lt" : 40
    }
}
```

`range` 过滤器既能包含也能排除范围，通过下面的选项：

* `gt`: `>` 大于
* `lt`: `<` 小于
* `gte`: `>=` 大于或等于
* `lte`: `<=` 小于或等于

下面是范围过滤器的一个示例：

```json
GET /my_store/products/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "range" : {
                    "price" : {
                        "gte" : 20,
                        "lt"  : 40
                    }
                }
            }
        }
    }
}
```

<!-- SENSE: 080_Structured_Search/25_Range_filter.json -->

假如你需要不设限的范围，去掉一边的限制就可以了：

```json
"range" : {
    "price" : {
        "gt" : 20
    }
}
```

<!-- SENSE: 080_Structured_Search/25_Range_filter.json -->

#### 日期范围

`range` 过滤器也可以用于日期字段：

```json
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-07 00:00:00"
    }
}
```

当用于日期字段时，`range` 过滤器支持_日期数学_操作。例如，我们想找到所有最近一个小时的文档：

```json
"range" : {
    "timestamp" : {
        "gt" : "now-1h"
    }
}
```

这个过滤器将始终能找出所有时间戳大于当前时间减 1 小时的文档，让这个过滤器像_移窗_一样通过你的文档。

日期计算也能用于实际的日期，而不是仅仅是一个像 now 一样的占位符。只要在日期后加上双竖线 `||`，就能使用日期数学表达式了。

```json
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-01 00:00:00||+1M" <1>
    }
}
```

<1> 早于 2014 年 1 月 1 号加一个月

日期计算是与_日历相关_的，所以它知道每个月的天数，每年的天数，等等。更详细的关于日期的信息可以在这里找到 [日期格式手册](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/mapping-date-format.html)

#### 字符串范围

`range` 过滤器也可以用于字符串。字符串范围根据_字典_或字母顺序来计算。例如，这些值按照字典顺序排序：

* 5, 50, 6, B, C, a, ab, abb, abc, b

提示：倒排索引中的短语按照字典顺序排序，也是为什么字符串范围使用这个顺序。

假如我们想让范围从 `a` 开始而不包含 `b`，我们可以用类似的 `range` 过滤器语法：

```json
"range" : {
    "title" : {
        "gte" : "a",
        "lt" :  "b"
    }
}
```

当心基数：

数字和日期字段的索引方式让他们在计算范围时十分高效。但对于字符串来说却不是这样。为了在字符串上执行范围操作，Elasticsearch 会在这个范围内的每个短语执行 `term` 操作。这比日期或数字的范围操作慢得多。

字符串范围适用于一个基数较小的字段，一个唯一短语个数较少的字段。你的唯一短语数越多，搜索就越慢。

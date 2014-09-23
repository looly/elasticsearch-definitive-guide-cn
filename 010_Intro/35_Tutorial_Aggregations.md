## 分析
最后，我们还有一个需求需要完成：允许管理者在职员目录中分析。
Elasticsearch把这项功能叫做**聚合(aggregations)**，它允许你在数据基础上生成复杂的统计。它很像SQL中的`GROUP BY`但是功能更强大。

举个例子，让我们找到最受员工欢迎的兴趣：

```Javascript
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

忽略语法只看结果：

```Javascript
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

We can see that two employees are interested in music, one in forestry and one
in sports.  These aggregations are not precalculated -- they are generated on
the fly from the documents which match the current query. If we want to know
the popular interests of people called ``Smith'', we can just add the
appropriate query into the mix:



[source,js]
--------------------------------------------------
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
--------------------------------------------------
// SENSE: 010_Intro/35_Aggregations.json

The `all_interests` aggregation has changed to include only documents matching our query:

[source,js]
--------------------------------------------------
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
--------------------------------------------------

Aggregations allow hierarchical rollups too.  For example, let's find the
average age of employees who share a particular interest:

[source,js]
--------------------------------------------------
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
--------------------------------------------------
// SENSE: 010_Intro/35_Aggregations.json

The aggregations that we get back are a bit more complicated, but still fairly
easy to understand:

[source,js]
--------------------------------------------------
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
--------------------------------------------------

The output is basically an enriched version of the first aggregation we ran.
We still have a list of interests and their counts, but now each interest has
an additional `avg_age` which shows the average age for all employees having
that interest.

Even if you don't understand the syntax yet, you can easily see how very
complex aggregations and groupings can be accomplished using this feature.
The sky is the limit as to what kind of data you can extract!

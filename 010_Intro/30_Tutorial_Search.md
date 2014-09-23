## 检索文档
现在我们的Elasticsearch中已经存储了一些数据，我们可以根据业务需求开始进行工作了。第一个需求是能够检索单个员工的信息。

这对于Elasticsearch非常简单。我们只要执行HTTP GET请求并指出文档的“地址”——索引、类型和ID既可。使用这三部分，就可以返回JSON原始文档：

```Jacscript
GET /megacorp/employee/1
```

响应的内容中包含一些文档的元数据信息，John Smith的原始JSON文档做为`_source`字段存在。

```Javascript
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

>同样的方式，我们通过把HTTP方法`PUT`改为`GET`来检索文档，我们可以使用`DELETE`方法删除文档，使用`HEAD`方法检查文档是否存在。如果为了替换已存在的文档，我们只需使用`PUT`方法即可。

## 简单搜索
`GET`请求确实足够简单——你想要什么，它就可以返回什么文档。让我们来看一些更复杂的东西，就像简单搜索！

我们尝试一个最简单的搜索全部员工的请求：

```Javascript
GET /megacorp/employee/_search
```

你可以看到我们依旧使用`megacorp`索引和`employee`类型，但是与使用指定ID的文档不同，现在使用`_search`端点。响应内容的`hits`数组中包含了所有的三个文档。一般情况下搜索会返回前10个结果。

```Javascript
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

>**注意**：

>响应内容不仅会告诉我们哪些文档被匹配，而且这些文档内容会被包含其中：所有的信息我们都需要显示搜索结果给用户。

接下来，让我们搜索姓氏中包含**“Smith”**的员工。要做到这一点，我们将在命令行中使用**“轻量级”**的搜索方法。这种方法常被称作**查询字符串(query string)**搜索，因为我们要通过URL参数来传递查询关键字：

```Javascript
GET /megacorp/employee/_search?q=last_name:Smith
```

我们依旧使用`_search`端点，然后将查询本身传递给参数`q=`。这样可以得到所有姓氏为Smith的结果：

```Javascript
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

## 使用DSL语句查询
查询字符串搜索是便于通过命令行完成**点对点(ad hoc)**的搜索，但是它也有局限性（参阅简单搜索章节）。Elasticsearch提供更加丰富且灵活的查询语言叫做**DSL查询(Query DSL)**,它允许你构建更加复杂、强大的搜索。

**DSL(Domain Specific Language领域特定语言)**指定JSON做为请求体。我们可以这样表示之前关于“Smith”的查询:

```Javascript
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

这会返回与之前查询相同的结果。你可以看到有些东西做了改变，我们不再使用**查询字符串(query string)**做为参数，而是使用请求体代替。这个请求体使用JSON表示，其中使用了`match`语句（查询类型之一，其余我们将在接下来的章节学习到）。

## 更复杂的搜索
我们让搜索变的复杂一些。我们依旧想要找到姓氏为“Smith”的员工，但是我们只想得到年龄大于30岁的员工。我们的语句将做一些改变用来添加**过滤器(filter)**,它允许我们有效的执行一个结构化搜索：

```Javascript
GET /megacorp/employee/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 } <1>
                }
            },
            "query" : {
                "match" : {
                    "last_name" : "smith" <2>
                }
            }
        }
    }
}
```

* <1> 这部分查询是`range`**过滤器(filter)**,它用于查找所有年龄大于30岁的数据（译者注：`age`字段大于30的数据），——`gt`代表"greater than"。
* <2> 这部分查询与之前的`match`**语句(query)**一致。

现在不要担心语法太多，我们将会在后面的章节详细的讨论。只要知道我们添加了一个**过滤器(filter)**用于执行区间搜索，然后重复利用了之前的`match`语句。现在我们只显示一个32岁且名字是“Jane Smith”的员工了：

```Javascript
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

## 全文搜索
到目前为止搜索都很简单：简单的名字，通过年龄筛选。让我们尝试一种更高级的搜索，全文搜索——一种传统数据库很难实现的功能。

我们将会搜索所有喜欢**“rock climbing”**的员工：

```Javascript
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

你可以看到我们使用与之前一致的`match`查询搜索`about`字段中的**"rock climbing"**，我们会得到两个匹配文档：

```Javascript
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, <1>
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, <1>
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```
- <1> 相关评分。

一般Elasticsearch根据相关评分排序，相关评分是根据文档与语句的匹配度来得出，第一个最高分很明确：John Smith的`about`字段明确的写到**“rock
climbing”**。

但是为什么Jane Smith也会出现在结果里？原因是**“rock”**在她的`abuot`字段中提及了。因为只有**“rock”**被提及而**“climbing”**没有，所以她的`_score`要低于John。

这个例子很好的解释了Elasticsearch如何进行全文字段搜索且首先返回相关性性最大的结果。**相关性(relevance)**概念在Elasticsearch中非常重要，而这也是它与传统关系型数据库中记录只有匹配和不匹配概念最大的不同。

## 短语搜索
能找到字段中单独的单次固然最好，但是有时候你想要匹配确切的单词序列或者**短语(phrases)**。例如我们想要查询`about`包含完整短语**“rock climbing”**的员工。

为了实现以上效果，我们将查询`match`变更为`match_phrase`:

```Javascript
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

毫无悬念返回John Smith的文档：

```Javascript
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         }
      ]
   }
}
```

## 高亮我们的搜索
很多应用喜欢从每个搜索结果中**高亮(highlight)**匹配到的关键字，以便用户可以知道**为什么**文档这样匹配查询。Elasticsearch中高亮片段是非常容易的。

让我们在之前的语句上增加`highlight`参数：

```Javascript
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

当我们运行这个语句，会命中与之前相同的结果，但是会得到一个新的叫做`highlight`的部分，这里包括了`about`字段中匹配的文本片段，并且用`<em></em>`包围匹配到的单词。

```Javascript
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" <1>
               ]
            }
         }
      ]
   }
}
```

<1> The highlighted fragment from the original text.
- <1> 原有文本中高亮的片段

你可以在高亮章节阅读更多关于搜索高亮的部分。

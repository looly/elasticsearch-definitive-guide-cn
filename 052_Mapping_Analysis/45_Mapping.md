## 映射

正如《数据吞吐》一节所说，索引中每个文档都有一个**类型(type)**。
每个类型拥有自己的**映射(mapping)**或者**模式定义(schema definition)**。一个映射定义了字段类型，每个字段的数据类型，以及字段被Elasticsearch处理的方式。映射还用于设置关联到类型上的元数据。

在《映射》章节我们将探讨映射的细节。这节我们只是带你入门。

### 核心简单字段类型

Elasticsearch支持以下简单字段类型：

|类型            |  表示的数据类型                    |
|----------------|------------------------------------|
|String          |  `string`                          |
|Whole number    |  `byte`, `short`, `integer`, `long`|
|Floating point  |  `float`, `double`                 |
|Boolean         |  `boolean`                         |
|Date            |  `date`                            |

当你索引一个包含新字段的文档——一个之前没有的字段——Elasticsearch将使用动态映射猜测字段类型，这类型来自于JSON的基本数据类型，使用以下规则：

|JSON type                          |          Field type    |
|-----------------------------------|------------------------|
|Boolean: `true` or `false`         |          `"boolean"`   |
|Whole number: `123`                |          `"long"`      |
|Floating point: `123.45`           |          `"double"`    |
|String, valid date: `"2014-09-15"` |          `"date"`      |
|String: `"foo bar"`                |          `"string"`    |

> ### 注意
> 这意味着，如果你索引一个带引号的数字——`"123"`，它将被映射为`"string"`类型，而不是`"long"`类型。然而，如果字段已经被映射为`"long"`类型，Elasticsearch将尝试转换字符串为long，并在转换失败时会抛出异常。

### 查看映射

我们可以使用`_mapping`后缀来查看Elasticsearch中的映射。在本章开始我们已经找到索引`gb`类型`tweet`中的映射：

```javascript
GET /gb/_mapping/tweet
```

这展示给了我们字段的映射（叫做**属性(properties)**），这些映射是Elasticsearch在创建索引时动态生成的：

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

> ### 小提示
> 错误的映射，例如把`age`字段映射为`string`类型而不是`integer`类型，会造成查询结果混乱。

> 要检查映射类型，而不是假设它是正确的！

### 自定义字段映射

映射中最重要的字段参数是`type`。除了`string`类型的字段，你可能很少需要映射其他的`type`：

```javascript
{
    "number_of_clicks": {
        "type": "integer"
    }
}
```

`string`类型的字段，默认的，考虑到包含全文本，它们的值在索引前要经过分析器分析，并且在全文搜索此字段前要把查询语句做分析处理。

对于`string`字段，两个最重要的映射参数是`index`和`analyer`。

### `index`

`index`参数控制字符串以何种方式被索引。它包含以下三个值当中的一个：

|值            |解释                                  |
|--------------|--------------------------------------|
|`analyzed`    |首先分析这个字符串，然后索引。换言之，以全文形式索引此字段。                                          |
|`not_analyzed`|索引这个字段，使之可以被搜索，但是索引内容和指定值一样。不分析此字段。                                |
|`no`          |不索引这个字段。这个字段不能为搜索到。|

`string`类型字段默认值是`analyzed`。如果我们想映射字段为确切值，我们需要设置它为`not_analyzed`：

```javascript
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}
```

> 其他简单类型——`long`、`double`、`date`等等——也接受`index`参数，但相应的值只能是`no`和`not_analyzed`，它们的值不能被分析。

### 分析

对于`analyzed`类型的字符串字段，使用`analyzer`参数来指定哪一种分析器将在搜索和索引的时候使用。默认的，Elasticsearch使用`standard`分析器，但是你可以通过指定一个内建的分析器来更改它，例如`whitespace`、`simple`或`english`。

```javascript
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```

在《自定义分析器》章节我们将告诉你如何定义和使用自定义的分析器。

### 更新映射

你可以在第一次创建索引的时候指定映射的类型。此外，你也可以晚些时候为新类型添加映射（或者为已有的类型更新映射）。

> ### 重要
> 你可以向已有映射中**增加**字段，但你不能**修改**它。如果一个字段在映射中已经存在，这可能意味着那个字段的数据已经被索引。如果你改变了字段映射，那已经被索引的数据将错误并且不能被正确的搜索到。

我们可以更新一个映射来增加一个新字段，但是不能把已有字段的类型那个从`analyzed`改到`not_analyzed`。

为了演示两个指定的映射方法，让我们首先删除索引`gb`：

```sh
DELETE /gb
```

然后创建一个新索引，指定`tweet`字段的分析器为`english`：

```javascript
PUT /gb <1>
{
  "mappings": {
    "tweet" : {
      "properties" : {
        "tweet" : {
          "type" :    "string",
          "analyzer": "english"
        },
        "date" : {
          "type" :   "date"
        },
        "name" : {
          "type" :   "string"
        },
        "user_id" : {
          "type" :   "long"
        }
      }
    }
  }
}
```
`<1>` 这将创建包含`mappings`的索引，映射在请求体中指定。

再后来，我们决定在`tweet`的映射中增加一个新的`not_analyzed`类型的文本字段，叫做`tag`，使用`_mapping`后缀:

```javascript
PUT /gb/_mapping/tweet
{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}
```

注意到我们不再需要列出所有的已经存在的字段，因为我们没法修改他们。我们的新字段已经被合并至存在的那个映射中。

### 测试映射

你可以通过名字使用`analyze` API测试字符串字段的映射。对比这两个请求的输出：

```javascript
GET /gb/_analyze?field=tweet
Black-cats <1>

GET /gb/_analyze?field=tag
Black-cats <1>
```

`<1>` 我们想要分析的文本被放在请求体中。

`tweet`字段产生两个词，`"black"`和`"cat"`,`tag`字段产生单独的一个词`"Black-cats"`。换言之，我们的映射工作正常。

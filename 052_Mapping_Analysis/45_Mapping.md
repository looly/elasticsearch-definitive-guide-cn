## 映射

在<<data-in-data-out>>一节中解释过，每个文档在一个索引中有一个_类型(type，从属于Indices)_。
每种类型有它自己的_映射(mapping)_或称之为_模式定义(schema definition)_。
一个映射以类型(type，从属于Indices)定义了所有字段，以数据类型定义了每一个字段，
并且字段如何为Elasticsearch所处理。一个映射也用于配置类型相关的元数据。

我们将在<<映射>>一章中详细讨论映射的细节。本节我们仅讨论足够使你入门的知识。

### 核心字段类型

Elasticsearch支持以下字段类型:


> String:         ::  `string`
> 
> Whole number:   ::  `byte`, `short`, `integer`, `long`
> 
> Floating point: ::  `float`, `double`
> 
> Boolean:        ::  `boolean`
> 
> Date:           ::  `date`

当你索引一个包含新字段的文档时(之前没见过的字段)，
Elasticsearch将使用动态映射(<<动态映射，070>>)在JSON中可用的基础数据类型猜测字段的类型。使用以下的规则：

> *JSON type:*                       ::          *Field type:*
>
> Boolean: `true` or `false`         ::          `"boolean"`
>
> Whole number: `123`                ::          `"long"`
>
> Floating point: `123.45`           ::          `"double"`
>
> String, valid date: `"2014-09-15"` ::          `"date"`
>
> String: `"foo bar"`                ::          `"string"`

注意：这意味着，如果你索引一个双引号中的数字，如`"123"`，这将被映射为`"string"`类型，而不是`"long"`类型。
然而，如果字段已被映射为`"long"`类型，Elasticsearch将尝试将string转换为long类型，如果无法转换则报错。

### 查看映射

我们可以使用`/_mapping`查看一个或多个索引(Indices)中的一个或多个类型(types)的映射。
在本章开始我们已经为索引`gb`的类型`tweet`设定了映射：

```javascript
GET /gb/_mapping/tweet
```

这向我们展现了Elasticsearch由我们索引的文档动态生成的字段的映射(被称为_properties_)：

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


****

###### TIP

不正确的映射，比如`age`字段被映射为`string`类型而非`integer`，会导致你查询结果的混乱。

不要认为你的映射是正确的，仔细检查！

****

### 自定义字段映射

一个字段最重要的属性就是`type`。对于非`string`字段，你除了`type`外很少需要映射其他的属性：

```javascript
{
    "number_of_clicks": {
        "type": "integer"
    }
}
```

默认情况下，类型为`"string"`的字段会被视为全文文本。
所以，它们的值将会在索引之前经过分词器的处理，
对于全文文本查询将会使得查询字串(query string)也被分词器所处理。

对于`string`字段有两个最重要的映射属性，它们是`index`和`analyzer`。

The two most important mapping attributes for `string` fields are
`index` and `analyzer`.

#### `index`

`index`属性控制了字串如何被索引。能使用如下值中的一种：

`analyzed`::        首先对字符串进行分词，然后索引。换句话说就是以全文文本进行索引。

`not_analyzed`::    索引这个字段，时期可以被查询，但是指定为确切值，也就是不要分词。

`no`::              不要索引这个字段，这个字段不能被查询。

`string`字段`index`的默认值为`analyzed`。如果我们想以确切值进行索引，需要设置为`not_analyzed`：

```javascript
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}
```


****

其他的类型(`long`，`double`，`date`等)也接受`index`参数，
但是仅允许`no`和`not_analyzed`，不能被分词。

****

#### `analyzer`

对于`analyzed`的string字段，使用`analyzer`属性以指定在索引时应用哪种分词器。
默认情况下，Elasticsearch使用`standard`分词器，但是你可以修改为一种内置分词器，
比如`whitespace`，`simple`或`english`：

```javascript
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```

在自定义分词器(<<自定义分词器， 070>>)一节中，将会展现如何定义并使用自定义的分词器。


### 更新映射

你可以使用`/_mapping`在创建索引时为一种类型指定映射。
或者，稍后为一种新类型增加映射(或为已有的类型更新映射)，

****

###### 重要

当你能为已存的映射*新增*时，你就不能*修改*它。
如果一个字段已存在于一种映射中，那么它很可能意味着字段中的数据已索引。
如果你想修改字段映射，已索引的数据将出错并且不会被正确的查询。

****

我们可以向一个映射添加一个新字段以更新它，但是不能修改已有的字段从`analyzed`到`not_analyzed`。

为了演示两种方式(新增映射，通过添加新字段修改映射)，让我们首先删除索引`gb`：

```shell
DELETE /gb
```

然后创建一个新的索引，指定`tweet`字段使用`english`分词器：

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

<1> 这将创建以`mappings`指定的映射创建一个索引。

稍后，我们决定向`tweet`映射新增一个新的`not_analyzed`文本字段`tag`，使用`_mapping`


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

注意我们不需要再次列出所有已存的字段，正如我们无法修改它们一样。
新字段会合并至已存的映射中。

### 映射测试

你可以使用`analyze` API测试指定名字的string字段的映射。比较下面两个请求的输出：

```javascript
GET /gb/_analyze?field=tweet
Black-cats <1>

GET /gb/_analyze?field=tag
Black-cats <1>
```

<1> 放置我们需要分词的的文本

`tweet`字段生成两个单词`"black"`和`"cat"`，而`tag`字段生成一个单词`"Black-cats"`。
换句话说，我们的映射正常工作了。

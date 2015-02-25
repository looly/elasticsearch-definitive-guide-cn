## 分词(Analysis)及分词器(analyzers)

_分词_是一种处理过程：

*  首先，标记(tokenization)一整块的文本为合适的独立_单词(terms)_以供倒排索引使用，
*  然后标准化(normalization)这些单词为标准形式以提升他们的``可搜索性``或_recall_。

这个任务是分词器(analyzers)完成的。一个_分词器_是一个框架，将三个功能打包为一个单一的包：

字符过滤器(Character filters)::

    首先，字符串按顺序通过_字符过滤器_。它们的任务是在标记(tokenization)前整理字符串。
    一个字符过滤器能剔除HTML标签，或转换`"&"`字符为`"and"`。

标记器(Tokenizer)::

    下一步，字符串被_标记器(tokenizer)_标记为独立的单词。
    一个简单的标记器会分割文本为一个个单词，分隔符可能是空格或标点符号。

标记过滤器(Token filters)::

    最后，每个单词会按顺序通过_标记过滤器_。他们可以改变单词(如：将`"Quick"`变全小写)，
    移除单词(如：禁用词`"a"`，`"and"`，`"the"`等)或增加单词(如：同义词类似于`"jump"`及`"leap"`)

Elasticsearch默认提供很多种字符过滤器，标记器和标记过滤器。
它们可以被组合以创建自定义分词器套件以满足不同需求。
我们将在<<自定义分词器>>中探讨更多的细节。

### 内建分词器

同时，Elasticsearch也提供了一些自带的分词器，你可以直接使用。
我们在下面列出了最重要的几个，并演示它们的差异。
我们列出下面字符串经过分词后的单词

    "Set the shape to semi-transparent by calling set_trans(5)"

标准分词器(Standard analyzer)::

标准分词器是Elasticsearch默认的分词器。他是分析任何语言最好的通用化选择。
它基于_单词边界(word boundaries)_(Unicode联盟所定义，参见http://www.unicode.org/reports/tr29/)分割文本，并且剔除大多数标点符号。
最后，它会变换所有的单词为小写形式，类似于这样：

    set, the, shape, to, semi, transparent, by, calling, set_trans, 5

简单分词器(Simple analyzer)::

简单分词器分割任何非字符的文本，并且变换所有单词为小写形式，类似于这样：

    set, the, shape, to, semi, transparent, by, calling, set, trans

空格分词器(Whitespace analyzer)::

空格分词器仅以空格分割文本，不做单词的小写化，类似于这样：

    Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

语言分词器(Language analyzers)::

语言特性分词器在很多语言中可用。它们会使用特定的语言特点。
例如，`english`分词器会以一些英语禁用词(对相关性不产生影响的常用词如`and`或`the`)
移除单词或提取词干，因为它理解英文语法。


`english`分词器将处理文本类似于这样：

    set, shape, semi, transpar, call, set_tran, 5

注意`"transparent"`，`"calling"`及`"set_trans"`是如何被提取为它们的词根形式的。

### 什么时候使用分词器

当我们*索引*一个文档时，它的全文字段将分词为创建倒排索引所用的单词。
然而，当我们在全文字段中*查询*时，我们需要以*相同的分词过程*传递查询字串(query string)，
以保证我们查询的单词和倒排索引中的单词是相同格式。

我们稍后将探讨的全文查询能够理解每个字段如何被定义的，所以查询时ES能正确处理：

 * 当查询一个*全文文本(full text)*字段时，ES对查询字符串(query string)应用相同的分词器

 * 当查询一个*确切值(exact value)*字段时，ES不会对查询字符串(query string)分词，而是使用你指定的确切值进行查询

现在你理解了我们在本章开头<<映射及分词>>所展示的查询为什么有这种结果：

* `date`字段包含一个确切值：单词`"2014-09-15"`。
* `_all`字段是一个全文文本字段，所以分词处理流程将把这个日期转换为三个单词：`"2014"`，`"09"`及`"15"`.

当我们在`_all`字段中查询`2014`时，匹配了12条推文，因为它们中都含有单词`2014`：

```shell
GET /_search?q=2014              # 12 个结果
```

当我们在`_all`字段中查询`2014-09-15`时，首先分词器把查询字串处理为匹配`2014`，`09`，`15`中*任意*单词的查询。
这也会返回12条推文，因为所有的结果都包含单词`2014`：

```shell
GET /_search?q=2014-09-15        # 还是 12 个结果 !
```

当我们在`date`字段中查询`2014-09-15`时，它就在查询*确切的*日期了，所以仅找到了一条推文：
```shell
GET /_search?q=date:2014-09-15   # 1  一个结果
```

当我们在`date`字段中查询`2014`，它找不到任何文档，因为没有文档包含这个确切值：

```shell
GET /_search?q=date:2014         # 0  个结果 !
```

### 测试分词器

你刚刚接触Elasticsearch时，有时很难去理解什么东西是实际标记并储存在你的索引中的。
为了更好理解到底发生了什么，你可以使用`analyze` API来观察文本是如何被分词的。
特别是应用于查询字符串(query string)参数的分词器，及内容中的文本是如何分词的：


```javascript
GET /_analyze?analyzer=standard
Text to analyze
```

结果中的每一个元素都代表着一个单词：

```javascript
{
   "tokens": [
      {
         "token":        "text",
         "start_offset": 0,
         "end_offset":   4,
         "type":         "<ALPHANUM>",
         "position":     1
      },
      {
         "token":        "to",
         "start_offset": 5,
         "end_offset":   7,
         "type":         "<ALPHANUM>",
         "position":     2
      },
      {
         "token":        "analyze",
         "start_offset": 8,
         "end_offset":   15,
         "type":         "<ALPHANUM>",
         "position":     3
      }
   ]
}
```


`token`是实际的单词，将被储存在索引(index而非Indices)中。
`position`指出出现在原始文本中的单词的顺序。
`start_offset`及`end_offset`指出原始词汇出现在原始字符串的字符位置。

`analyze` API是一个理解Elasticsearch索引(indices)机理的非常有用的工具，我们会后面会讨论更多。

### 指定分词器

当Elasticsearch在文档中检测到一个新的string字段，它会自动配置此字段为全文文本`string`字段并且用`standard`分词器进行分词。

你并不是总希望如此。也许你想要对你的数据应用不同的分词器以更好的适应它的语言。
而有时候你想要string字段仅仅是一个string字段 - 作为确切值进行索引，不要进行分词，比如用户ID或内部状态字段或tag。

为了达成这些目的，我们必须用_mapping_手动配置这些字段。

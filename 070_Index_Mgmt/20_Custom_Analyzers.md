### 自定义分析器

虽然 Elasticsearch 内置了一系列的分析器，但是真正的强大之处在于定制你自己的分析器。你可以通过在配置文件中组合字符过滤器，分词器和标记过滤器，来满足特定数据的需求。

在 【分析器介绍】 中，我们提到 _分析器_ 是三个顺序执行的组件的结合（字符过滤器，分词器，标记过滤器）。

字符过滤器

> 字符过滤器是让字符串在被分词前变得更加“整洁”。例如，如果我们的文本是 HTML 格式，它可能会包含一些我们不想被索引的 HTML 标签，诸如 `<p>` 或 `<div>`。

> 我们可以使用 [`html_strip` 字符过滤器](http://bit.ly/1B6f4Ay) 来删除所有的 HTML 标签，并且将 HTML 实体转换成对应的 Unicode 字符，比如将 `&Aacute;` 转成 `Á`。

> 一个分析器可能包含零到多个字符过滤器。

分词器

> 一个分析器 _必须_ 包含一个分词器。分词器将字符串分割成单独的词（terms）或标记（tokens）。`standard` 分析器使用 [`standard` 分词器](http://bit.ly/1E3Fd1b)将字符串分割成单独的字词，删除大部分标点符号，但是现存的其他分词器会有不同的行为特征。

> 例如，[`keyword` 分词器](http://bit.ly/1ICd585)输出和它接收到的相同的字符串，不做任何分词处理。[`whitespace` 分词器]只通过空格来分割文本。[`pattern` 分词器]可以通过正则表达式来分割文本。

标记过滤器

> 分词结果的 _标记流_ 会根据各自的情况，传递给特定的标记过滤器。

> 标记过滤器可能修改，添加或删除标记。我们已经提过 [`lowercase`](http://bit.ly/1DIeXvZ) 和 [`stop`](http://bit.ly/1INX4tN) 标记过滤器，但是 Elasticsearch 中有更多的选择。[`stemmer` 标记过滤器](http://bit.ly/1AUfpDN)将单词转化为他们的根形态（root form）。[`ascii_folding` 标记过滤器](http://bit.ly/1ylU7Q7)会删除变音符号，比如从 `très` 转为 `tres`。 [`ngram`](http://bit.ly/1CbkmYe) 和 [`edge_ngram`](http://bit.ly/1DIf6j5) 可以让标记更适合特殊匹配情况或自动完成。

在【深入搜索】中，我们将举例介绍如何使用这些分词器和过滤器。但是首先，我们需要阐述一下如何创建一个自定义分析器

### 创建自定义分析器

与索引设置一样，我们预先配置好 `es_std` 分析器，我们可以再 `analysis` 字段下配置字符过滤器，分词器和标记过滤器：

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": { ... custom character filters ... },
            "tokenizer":   { ...    custom tokenizers     ... },
            "filter":      { ...   custom token filters   ... },
            "analyzer":    { ...    custom analyzers      ... }
        }
    }
}
```

作为例子，我们来配置一个这样的分析器：

1. 用 `html_strip` 字符过滤器去除所有的 HTML 标签

2. 将 `&` 替换成 `and`，使用一个自定义的 `mapping` 字符过滤器

```
"char_filter": {
    "&_to_and": {
        "type":       "mapping",
        "mappings": [ "&=> and "]
    }
}
```

3. 使用 `standard` 分词器分割单词

4. 使用 `lowercase` 标记过滤器将词转为小写

5. 用 `stop` 标记过滤器去除一些自定义停用词。

```
"filter": {
    "my_stopwords": {
        "type":        "stop",
        "stopwords": [ "the", "a" ]
    }
}
```

根据以上描述来将预定义好的分词器和过滤器组合成我们的分析器：

```
"analyzer": {
    "my_analyzer": {
        "type":           "custom",
        "char_filter":  [ "html_strip", "&_to_and" ],
        "tokenizer":      "standard",
        "filter":       [ "lowercase", "my_stopwords" ]
    }
}
```

用下面的方式可以将以上请求合并成一条：

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
```

<!-- SENSE: 070_Index_Mgmt/20_Custom_analyzer.json -->

创建索引后，用 `analyze` API 来测试新的分析器：

```
GET /my_index/_analyze?analyzer=my_analyzer
The quick & brown fox
```

<!-- SENSE: 070_Index_Mgmt/20_Custom_analyzer.json -->

下面的结果证明我们的分析器能正常工作了：

```
{
  "tokens" : [
      { "token" :   "quick",    "position" : 2 },
      { "token" :   "and",      "position" : 3 },
      { "token" :   "brown",    "position" : 4 },
      { "token" :   "fox",      "position" : 5 }
    ]
}
```

除非我们告诉 Elasticsearch 在哪里使用，否则分析器不会起作用。我们可以通过下面的映射将它应用在一个 `string` 类型的字段上：

```
PUT /my_index/_mapping/my_type
{
    "properties": {
        "title": {
            "type":      "string",
            "analyzer":  "my_analyzer"
        }
    }
}
```

<!-- SENSE: 070_Index_Mgmt/20_Custom_analyzer.json -->

## 多字段字符串排序

`analyzed`的字符串字段同时也是多值字段，可是在这些字段上进行排序往往得不到你想要的值。
比如你分析一个字符 `"fine old art"`,它最终会得到三个值。假如我们想要按照第一个词首字母排序，
第一个单词相同的话，再用第二个词的首字母排，以此类推，可惜 ElasticSearch 在进行排序时
是得不到这些信息的。

当然你可以使用 `min` 和 `max` 模式来排（默认使用的是 `min` 模式）但它的依据是`art` 或者 `old`，
而不是我们所期望的那样。

为了使一个string字段可以进行排序，它必须只包含一个词：即完整的`not_analyzed`字符串。
当然我们需要对字段进行全文本搜索的时候还必须使用 `analyzed`。

在 `_source` 下相同的字符串上排序两次会造成不必要的资源浪费。
而我们想要的是一个字段中同时包含这两种索引方式。
现在我们介绍一个在所有核心字段类型上通用的参数 `fields`，这样我们就可以改变它的mapping：

```Javascript
"tweet": {
    "type":     "string",
    "analyzer": "english"
}
```

改变后的多值字段mapping如下：

```Javascript
"tweet": { <1>
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": { <2>
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}
```

<1>  `tweet` 字段用于全文本的 `analyzed` 索引方式不变。
<2>  新增的 `tweet.raw` 子字段索引方式是 `not_analyzed`。

现在，在给数据重建索引后，我们既可以使用 `tweet` 字段进行全文本搜索，也可以用`tweet.raw`字段进行排序：

```Javascript
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    },
    "sort": "tweet.raw"
}
```

>**警告**：
>对 `analyzed` 字段进行强制排序会消耗大量内存。
>详情请查阅《字段类型简介》相关内容。

<!--秀川译-->
### 分析控制

查询只能查找在倒排索引中出现的词，所以确保在文档索引的时候以及字符串查询的时候使用同一个分析器是很重要的，为了查询的词能够在倒排索引中匹配到。

尽管我们说文档中每个字段的分析器是已经定好的。但是字段可以有不同的分析器，通过给那个字段配置一个指定的分析器或者直接使用类型，索引，或节点上的默认分析器。在索引的时候，一个字段的值会被配置的或者默认的分析器分析。

举个例子，让我们添加一个字段到`my_index`：

```Javascript
PUT /my_index/_mapping/my_type
{
    "my_type": {
        "properties": {
            "english_title": {
                "type":     "string",
                "analyzer": "english"
            }
        }
    }
}
```

现在我们可以通过`analyze`API分析`Foxes`词来比较`english_title`字段中的值以及`title`字段中的值在创建索引的时候是怎么被分析的：

```Javacript
GET /my_index/_analyze?field=my_type.title   <1>
Foxes

GET /my_index/_analyze?field=my_type.english_title <2>
Foxes
```

<1> 使用`standard`分析器的`title`字段将会返回词`foxes`。

<2> 使用`english`分析器的`english_title`字段将会返回词`fox`。

这意味着，如果我们为精确的词`fox`执行一个低级别的`term`查询，`english_title`字段会匹配而`title`字段不会。

像`match`查询一样的高级别的查询可以知道字段的映射并且能够在被查询的字段上使用正确的分析器。我们可以在`validate-query` API的执行中看到这个：

```Javacript
GET /my_index/my_type/_validate/query?explain
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title":         "Foxes"}},
                { "match": { "english_title": "Foxes"}}
            ]
        }
    }
}
```

它会返回`explanation`：

    (title:foxes english_title:fox)

`match`查询为每个字段使用合适的分析器用来确保在那个字段上可以用正确的格式查询这个词。

#### 默认分析器

虽然我们可以在字段级别指定一个分析器，但是如果我们在字段级别没有指定分析器的话，那要怎么决定某个字段使用什么分析器呢？

分析器可以在好几个地方设置。Elasticsearch会查找每个级别直到找到它可以使用的分析器。在创建索引的时候，Elasticsearch查找分析器的顺序如下：

* 在映射文件中指定字段的`analyzer`，或者
* *在文档的`_analyzer`字段上指定分析器，或者*
* 在映射文件中指定类型的默认分析器`analyzer`
* 在索引映射文件中设置默认分析器`default`
* 在节点级别设置默认分析器`default`
* `standard`分析器

查找索引的时候，Elasticsearch查找分析器的顺序稍微有点不一样：

* *在查询参数中指定`analyzer`，或者*
* 在映射文件中指定字段的`analyzer`，或者
* 在映射文件中指定类型的默认分析器`analyzer`
* 在索引映射文件中设置默认分析器`default`
* 在节点级别设置默认分析器`default`
* `standard`分析器

> 提示：
>
> 上面列表中用斜体字的两行突出了创建索引以及查询索引的时候Elasticsearch查找分析器的区别。`_analyzer`字段允许你为每个文档指定默认的分析器(比如, english, french, spanish)，虽然在查询的时候指定`analyzer`参数，但是在一个索引中处理多种语言这并不是一个好方法，因为在多语言环境下很多问题会暴露出来。

有时候，在创建索引与查询索引的时候使用不同的分析器也是有意义的。举个例子：在创建索引的时候想要索引同义词 (比如, 出现quick的时候，我们也索引 fast, rapid, 和 speedy)。但是在查询索引的时候，我们不需要查询所有的同义词，我们只要查询用户输入的一个单词就可以了，它可以是`quick`,
`fast`, `rapid`, 或者 `speedy`。

为了满足这种差异，Elasticsearch也支持`index_analyzer` 和 `search_analyzer` 参数，并且分析器被命名为`default_index`和`default_search`。

把这些额外的参数考虑进去，Elasticsearch查找所有的分析器的顺序实际上像下面的样子：

* 在映射文件中指定字段的`index_analyzer`，或者
* 在映射文件中指定字段的`analyzer`，或者
* 在文档的`_analyzer`字段上指定分析器，或者
* 在映射文件中指定类型的创建索引的默认分析器`index_analyzer`
* 在映射文件中指定类型的默认分析器`analyzer`
* 在索引映射文件中设置创建索引的默认分析器`default_index`
* 在索引映射文件中设置默认分析器`default`
* 在节点级别设置创建索引的默认分析器`default_index`
* 在节点级别设置默认分析器`default`
* `standard`分析器

以及查询索引的时候:

* 在查询参数中指定`analyzer`，或者
* 在映射文件中指定字段的`search_analyzer`，或者
* 在映射文件中指定字段的`analyzer`，或者
* 在映射文件中指定类型的查询索引的默认分析器`analyzer`
* 在索引映射文件中设置查询索引的默认分析器`default_search`
* 在索引映射文件中设置默认分析器`default_search`
* 在节点级别设置查询索引的默认分析器`default_search`
* 在节点级别设置默认分析器`default`
* `standard` 分析器

#### 实际配置分析器

可以指定分析器的地方实在是太多了，但实际上，指定分析器很简单。

#### 用索引配置，而不是用配置文件

第一点要记住的是，尽管你开始使用Elasticsearch仅仅只是为了一个简单的目的或者是一个应用比如日志，但很可能你会发现更多的案例（use cases翻译成案例不知道合不合适，如果有更好的用词，请联系我，Tks），结局是在同一个集群中运行着好几个不同的应用。每一个索引都需要是独立的，并且可以被独立的配置。你不要想着给一个案例设置默认值，但是不得不重写他们来适配后面的案例。

这个规则把节点级别的配置分析器方法排除在外了。另外，节点级别的分析器配置方法需要改变每个节点的配置文件并且重启每个节点，这将成为维护的噩梦。保持Elasticsearch持续运行并通过API来管理索引的设置是一个更好的方法。

#### 保持简便性

大多数时间，你可以预先知道文档会包含哪些字段。最简单的方法是在你创建索引或者添加类型映射的时候为每一个全文检索字段设置分析器。虽然这个方法有点啰嗦，但是它可以很容易的看到哪个字段应用了哪个分析器。

通常情况下，大部分的字符串字段是确切值`not_analyzed`字段（索引但不分析字段）比如标签，枚举，加上少数全文检索字段会使用默认的分析器，像`standard` 或者 `english` 或者其他语言。然后你可能只有一到两个字段需要定制分析：或许`title`字段需要按照你查找的方式去索引来支持你的查找。（指的是你查找的字符串用什么分析器，创建索引就用什么分析器）

你可以在索引设置`default`分析器的地方为几乎所有全文检索字段设置成你想要的分析器，并且只要在一到两个字段指定专门的分析器。如果，在你的模型中，你每个类型都需要不同的分析器，那么在类型级别使用`analyzer`配置来代替。

> 提示：

> 一个普通的像日志一样的基于时间轴的工作流数据每天都得创建新的索引，忙着不断的创建索引。虽然这种工作流阻止你预先创建索引，但是你可以使用索引模板来指定新的索引的配置和映射。

<!--
=== Controlling Analysis

Queries can find only terms that actually ((("full text search", "controlling analysis")))((("analysis", "controlling")))exist in the inverted index, so it
is important to ensure that the same analysis process is applied both to the
document at index time, and to the query string at search time so that the
terms in the query match the terms in the inverted index.

Although we say _document_, analyzers are determined per field.((("analyzers", "determined per-field"))) Each
field can have a different analyzer, either by configuring a specific analyzer
for that field or by falling back on the type, index, or node defaults.  At
index time, a field's value is analyzed by using the configured or default
analyzer for that field.

For instance, let's add a new field to `my_index`:

[source,js]
--------------------------------------------------
PUT /my_index/_mapping/my_type
{
    "my_type": {
        "properties": {
            "english_title": {
                "type":     "string",
                "analyzer": "english"
            }
        }
    }
}
--------------------------------------------------
// SENSE: 100_Full_Text_Search/30_Analysis.json

Now we can compare how values in the `english_title` field and the `title` field are
analyzed at index time by using the `analyze` API to analyze the word `Foxes`:

[source,js]
--------------------------------------------------
GET /my_index/_analyze?field=my_type.title   <1>
Foxes

GET /my_index/_analyze?field=my_type.english_title <2>
Foxes
--------------------------------------------------
// SENSE: 100_Full_Text_Search/30_Analysis.json

<1> Field `title`, which uses the default `standard` analyzer, will return the
    term `foxes`.

<2> Field `english_title`, which uses the `english` analyzer, will return the term
    `fox`.

This means that, were we to run a low-level `term` query for the exact term
`fox`, the `english_title` field would match but the `title` field would
not.

High-level queries like the `match` query understand field mappings and can
apply the correct analyzer for each field being queried.((("match query", "applying appropriate analyzer to each field"))) We can see this
in action with ((("validate query API")))the `validate-query` API:


[source,js]
--------------------------------------------------
GET /my_index/my_type/_validate/query?explain
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title":         "Foxes"}},
                { "match": { "english_title": "Foxes"}}
            ]
        }
    }
}
--------------------------------------------------
// SENSE: 100_Full_Text_Search/30_Analysis.json

which returns this `explanation`:

    (title:foxes english_title:fox)

The `match` query uses the appropriate analyzer for each field to ensure
that it looks for each term in the correct format for that field.

==== Default Analyzers

While we can specify an analyzer at the field level,((("full text search", "controlling analysis", "default analyzers")))((("analyzers", "default"))) how do we determine which
analyzer is used for a field if none is specified at the field level?

Analyzers can be specified at several levels.  Elasticsearch works through
each level until it finds an analyzer that it can use.  At index time, the
order ((("indexing", "applying analyzers")))is as follows:

* The `analyzer` defined in the field mapping, else
* _The analyzer defined in the `_analyzer` field of the document, else_
* The default `analyzer` for the `type`, which defaults to
* The analyzer named `default` in the index settings, which defaults to
* The analyzer named `default` at node level, which defaults to
* The `standard` analyzer

At search time, the ((("searching", "applying analyzers")))sequence is slightly different:

* _The `analyzer` defined in the query itself, else_
* The `analyzer` defined in the field mapping, else
* The default `analyzer` for the `type`, which defaults to
* The analyzer named `default` in the index settings, which defaults to
* The analyzer named `default` at node level, which defaults to
* The `standard` analyzer

[NOTE]
====
The two lines in italics in the preceding lists highlight differences in the index time sequence and the search time sequence.  The `_analyzer` field allows you to specify a default analyzer for each document (for example, `english`, `french`, `spanish`) while the `analyzer` parameter in the query specifies which analyzer to use on the query string. However, this is not the best way to handle multiple languages
in a single index because of the pitfalls highlighted in <<languages>>.
====

Occasionally, it makes sense to use a different analyzer at index and search
time.((("analyzers", "using different analyzers at index and search time"))) For instance, at index time we may want to index synonyms (for example, for every
occurrence of `quick`, we also index `fast`, `rapid`, and `speedy`). But at
search time, we don't need to search for all of these synonyms.  Instead we
can just look up the single word that the user has entered, be it `quick`,
`fast`, `rapid`, or `speedy`.

To enable this distinction, Elasticsearch also supports ((("index_analyzer parameter")))((("search_analyzer parameter")))the `index_analyzer`
and `search_analyzer` parameters, and((("default_search parameter"))) ((("default_index analyzer")))analyzers named `default_index` and
`default_search`.

Taking these extra parameters into account, the _full_ sequence at index time
really looks like this:

* The `index_analyzer` defined in the field mapping, else
* The `analyzer` defined in the field mapping, else
* The analyzer defined in the `_analyzer` field of the document, else
* The default `index_analyzer` for the `type`, which defaults to
* The default `analyzer` for the `type`, which defaults to
* The analyzer named `default_index` in the index settings, which defaults to
* The analyzer named `default` in the index settings, which defaults to
* The analyzer named `default_index` at node level, which defaults to
* The analyzer named `default` at node level, which defaults to
* The `standard` analyzer

And at search time:

* The `analyzer` defined in the query itself, else
* The `search_analyzer` defined in the field mapping, else
* The `analyzer` defined in the field mapping, else
* The default `search_analyzer` for the `type`, which defaults to
* The default `analyzer` for the `type`, which defaults to
* The analyzer named `default_search` in the index settings, which defaults to
* The analyzer named `default` in the index settings, which defaults to
* The analyzer named `default_search` at node level, which defaults to
* The analyzer named `default` at node level, which defaults to
* The `standard` analyzer

==== Configuring Analyzers in Practice

The sheer number of places where you can specify an analyzer is quite
overwhelming.((("full text search", "controlling analysis", "configuring analyzers in practice")))((("analyzers", "configuring in practice")))  In practice, though, it is pretty simple.

===== Use index settings, not config files

The first thing to remember is that, even though you may start out using
Elasticsearch for a single purpose or a single application such as logging,
chances are that you will find more use cases and end up running several
distinct applications on the same cluster.  Each index needs to be independent
and independently configurable. You don't want to set defaults for one use
case, only to have to override them for another use case later.

This rules out configuring analyzers at the node level.  Additionally,
configuring analyzers at the node level requires changing the config file on every
node and restarting every node, which becomes a maintenance nightmare. It's a
much better idea to keep Elasticsearch running and to manage settings only via
the API.

===== Keep it simple

Most of the time, you will know what fields your documents will contain ahead
of time.  The simplest approach is to set the analyzer for each full-text
field when you create your index or add type mappings.  While this approach is
slightly more verbose, it enables you to easily see which analyzer is being applied
to each field.

Typically, most of your string fields will be exact-value `not_analyzed`
fields such as tags or enums, plus a handful of full-text fields that will
use some default analyzer like `standard` or `english` or some other language.
Then you may have one or two fields that need custom analysis: perhaps the
`title` field needs to be indexed in a way that supports _find-as-you-type_.

You can set the `default` analyzer in the index to the analyzer you want to
use for almost all full-text fields, and just configure the specialized
analyzer on the one or two fields that need it.  If, in your model, you need
a different default analyzer per type, then use the type level `analyzer`
setting instead.

[NOTE]
====
A common work flow for time based data like logging is to create a new index
per day on the fly by just indexing into it.  While this work flow prevents
you from creating your index up front, you can still use 
http://bit.ly/1ygczeq[index templates]
to specify the settings and mappings that a new index should have.
====
-->

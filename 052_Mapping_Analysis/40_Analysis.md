## 分析和分析器

**分析(analysis)**是这样一个过程：

* 首先，表征化一个文本块为适用于倒排索引单独的**词(term)**
* 然后标准化这些词为标准形式，提高它们的“可搜索性”或“查全率”

这个工作是**分析器(analyzer)**完成的。一个**分析器(analyzer)**只是一个包装用于将三个功能放到一个包里：

### 字符过滤器

首先字符串经过**字符过滤器(character filter)**，它们的工作是在表征化（译者注：这个词叫做断词更合适）前处理字符串。字符过滤器能够去除HTML标记，或者转换`"&"`为`"and"`。

### 分词器

下一步，**分词器(tokenizer)**被表征化（断词）为独立的词。一个简单的**分词器(tokenizer)**可以根据空格或逗号将单词分开（译者注：这个在中文中不适用）。

### 表征过滤

最后，每个词都通过所有**表征过滤(token filters)**，它可以修改词（例如将`"Quick"`转为小写），去掉词（例如停用词像`"a"`、`"and"``"the"`等等），或者增加词（例如同义词像`"jump"`和`"leap"`）

Elasticsearch provides many character filters, tokenizers and token filters
out of the box. These can be combined to create custom analyzers suitable
for different purposes. We will discuss these in detail in <<custom-analyzers>>.

==== Built-in analyzers

However, Elasticsearch also ships with a number of pre-packaged analyzers that
you can use directly. We list the most important ones below and, to demonstrate
the difference in behaviour, we show what terms each would produce
from this string:

    "Set the shape to semi-transparent by calling set_trans(5)"


Standard analyzer::

The standard analyzer is the default analyzer that Elasticsearch uses. It is
the best general choice for analyzing text which may be in any language. It
splits the text on _word boundaries_, as defined by the
http://www.unicode.org/reports/tr29/[Unicode Consortium], and removes most
punctuation. Finally, it lowercases all terms. It would produce:
+
    set, the, shape, to, semi, transparent, by, calling, set_trans, 5

Simple analyzer::

The simple analyzer splits the text on anything that isn't a letter,
and lowercases the terms. It would produce:
+
    set, the, shape, to, semi, transparent, by, calling, set, trans

Whitespace analyzer::

The whitespace analyzer splits the text on whitespace. It doesn't
lowercase. It would produce:
+
    Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

Language analyzers::

Language-specific analyzers are available for many languages. They are able to
take the peculiarities of the specified language into account. For instance,
the `english` analyzer comes with a set of English stopwords -- common words
like `and` or `the` which don't have much impact on relevance -- which it
removes, and it is able to _stem_ English words because it understands the
rules of English grammar.
+
The `english` analyzer would produce the following:
+
    set, shape, semi, transpar, call, set_tran, 5
+
Note how `"transparent"`, `"calling"`, and `"set_trans"` have been stemmed to
their root form.

==== When analyzers are used

When we *index* a document, its full text fields are analyzed into terms which
are used to create the inverted index.  However, when we *search* on a full
text field,  we need to pass the query string through the *same analysis
process*, to ensure that we are searching for terms in the same form as those
that exist in the index.

Full text queries, which we will discuss later, understand how each field is
defined, and so they can do the right thing:

 * When you query a *full text* field, the query will apply the same analyzer
   to the query string to produce the correct list of terms to search for.

 * When you query an *exact value* field, the query will not analyze the
   query string, but instead search for the exact value that you have
   specified.

Now you can understand why the queries that we demonstrated at the
<<mapping-analysis,start of this chapter>> return what they do:

* The `date` field contains an exact value: the single term `"2014-09-15"`.
* The `_all` field is a full text field, so the analysis process has
  converted the date into the three terms: `"2014"`, `"09"` and `"15"`.

When we query the `_all` field for `2014`, it matches all 12 tweets, because
all of them contain the term `2014`:

[source,sh]
--------------------------------------------------
GET /_search?q=2014              # 12 results
--------------------------------------------------
// SENSE: 052_Mapping_Analysis/25_Data_type_differences.json

When we query the `_all` field for `2014-09-15`, it first analyzes the query
string to produce a query which matches *any* of the terms `2014`, `09` or
`15`. This also matches all 12 tweets, because all of them contain the term
`2014`:

[source,sh]
--------------------------------------------------
GET /_search?q=2014-09-15        # 12 results !
--------------------------------------------------
// SENSE: 052_Mapping_Analysis/25_Data_type_differences.json

When we query the `date` field for `2014-09-15`, it looks for that *exact*
date, and finds one tweet only:

[source,sh]
--------------------------------------------------
GET /_search?q=date:2014-09-15   # 1  result
--------------------------------------------------
// SENSE: 052_Mapping_Analysis/25_Data_type_differences.json

When we query the `date` field for `2014`, it finds no documents
because none contain that exact date:

[source,sh]
--------------------------------------------------
GET /_search?q=date:2014         # 0  results !
--------------------------------------------------
// SENSE: 052_Mapping_Analysis/25_Data_type_differences.json

[[analyze-api]]
==== Testing analyzers

Especially when you are new to Elasticsearch, it is sometimes difficult to
understand what is actually being tokenized and stored into your index.  To
better understand what is going on, you can use the `analyze` API to see how
text is analyzed. Specify which analyzer to use in the query string
parameters,  and the text to analyze in the body:

[source,js]
--------------------------------------------------
GET /_analyze?analyzer=standard
Text to analyze
--------------------------------------------------
// SENSE: 052_Mapping_Analysis/40_Analyze.json


Each element in the result represents a single term:

[source,js]
--------------------------------------------------
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
--------------------------------------------------

The `token` is the actual term that will be stored in the index. The
`position` indicates the order in which the terms appeared in the original
text. The `start_offset` and `end_offset` indicate the character positions
that the original word occupied in the original string.

The `analyze` API is a really useful tool for understanding what is happening
inside Elasticsearch indices, and we will talk more about it as we progress.

==== Specifying analyzers

When Elasticsearch detects a new string field in your documents, it
automatically configures it as a full text `string` field and analyzes it with
the `standard` analyzer.

You don't always want this. Perhaps you want to apply a different analyzer
which suits the language your data is in. And sometimes you want a
string field to be just a string field -- to index the exact value that
you pass in, without any analysis, such as a string user ID or an
internal status field or tag.

In order to achieve this, we have to configure these fields manually
by specifying the _mapping_.

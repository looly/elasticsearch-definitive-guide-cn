### 单一查询字符串(Single Query String)

bool查询是多字段查询的中流砥柱。在很多场合下它都能很好地工作，特别是当你能够将不同的查询字符串映射到不同的字段时。

问题在于，现在的用户期望能够在一个地方输入所有的搜索词条，然后应用能够知道如何为他们得到正确的结果。所以当我们把含有多个字段的搜索表单称为高级搜索(Advanced Search)时，是有一些讽刺意味的。高级搜索虽然对用户而言会显得更"高级"，但是实际上它的实现方式更简单。

对于多词，多字段查询并没有一种万能(one-size-fits-all)的方法。要得到最佳的结果，你需要了解你的数据以及如何使用恰当的工具。


#### 了解你的数据

当用户的唯一输入就是一个查询字符串时，你会经常碰到以下三种情况：

##### 1.最佳字段(Best fields)::

当搜索代表某些概念的单词时，例如"brown fox"，几个单词合在一起表达出来的意思比单独的单词更多。类似title和body的字段，尽管它们是相关联的，但是也是互相竞争着的。文档在相同的字段中应该有尽可能多的单词(译注：搜索的目标单词)，文档的分数应该来自拥有最佳匹配的字段。

##### 2.多数字段(Most fields)::

一个用来调优相关度的常用技术是将相同的数据索引到多个字段中，每个字段拥有自己的分析链(Analysis Chain)。

主要字段会含有单词的词干部分，同义词和消除了变音符号的单词。它用来尽可能多地匹配文档。

相同的文本可以被索引到其它的字段中来提供更加精确的匹配。一个字段或许会包含未被提取成词干的单词，另一个字段是包含了变音符号的单词，第三个字段则使用shingle来提供关于[单词邻近度(Word Proximity)](http://blog.csdn.net/dm_vincent/article/details/41800351)的信息。

以上这些额外的字段扮演者signal的角色，用来增加每个匹配的文档的相关度分值。越多的字段被匹配则意味着文档的相关度越高。

##### 3.跨字段(Cross fields)::

对于一些实体，标识信息会在多个字段中出现，每个字段中只含有一部分信息：

* Person: `first_name` 和 `last_name`
* Book: `title`, `author`, 和 `description`
* Address:  `street`, `city`, `country`, 和 `postcode`

此时，我们希望在任意字段中找到尽可能多的单词。我们需要在多个字段中进行查询，就好像这些字段是一个字段那样。

以上这些都是多词，多字段查询，但是每种都需要使用不同的策略。我们会在本章剩下的部分解释每种策略。


<!-- === Single Query String

The `bool` query is the mainstay of multiclause queries.((("multifield search", "single query string"))) It works well
for many cases, especially when you are able to map different query strings to
individual fields.

The problem is that, these days, users expect to be able to type all of their
search terms into a single field, and expect that the application will figure out how
to give them the right results.  It is ironic that the multifield search form
is known as _Advanced Search_&#x2014;it may appear advanced to the user, but it is
much simpler to implement.

There is no simple _one-size-fits-all_ approach to multiword, multifield
queries.  To get the best results, you have to _know your data_ and know how
to use the appropriate tools.

[[know-your-data]]
==== Know Your Data

When your only user input is a single query string, you will encounter three scenarios frequently:

Best fields::

When searching for words that represent a concept, such as ``brown fox,'' the
words mean more together than they do individually. Fields like the `title`
and `body`, while related, can be considered to be in competition with each
other. Documents should have as many words as possible in _the same field_,
and the score should come from the _best-matching field_.

Most fields::
+
--
A common technique for fine-tuning relevance is to index the same data into
multiple fields, each with its own analysis chain.

The main field may contain words in their stemmed form, synonyms, and words
stripped of their _diacritics_, or accents. It is used to match as many
documents as possible.

The same text could then be indexed in other fields to provide more-precise
matching.  One field may contain the unstemmed version, another the original
word with accents, and a third might use _shingles_ to provide information
about <<proximity-matching,word proximity>>.

These other fields act as _signals_ to increase the relevance score of each
matching document. The _more fields that match_, the better.
--

Cross fields::
+
--
For some entities, the identifying information is spread across multiple
fields, each of which contains just a part of the whole:

* Person: `first_name` and `last_name`
* Book: `title`, `author`, and `description`
* Address:  `street`, `city`, `country`, and `postcode`

In this case, we want to find as many words as possible in _any_ of the listed
fields. We need to search across multiple fields as if they were one big
field.
--

All of these are multiword, multifield queries, but each requires a
different strategy. We will examine each strategy in turn in the rest of this
chapter.
-->

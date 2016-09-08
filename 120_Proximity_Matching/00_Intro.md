<!--
[[proximity-matching]]
== Proximity Matching
translated by Yang
-->
## 模糊匹配

<!--
Standard full-text search with TF/IDF treats documents, or at least each field
within a document, as a big _bag of words_.((("proximity matching")))  The `match` query can tell us whether
that bag contains our search terms, but that is only part of the story.
It can't tell us anything about the relationship between words.
-->
一般的全文检索方式使用 TF/IDF 处理文本或者文本数据中的某个字段内容。将字面切分成很多字、词(word)建立索引，match查询用query中的term来匹配索引中的字、词。match查询提供了文档数据中是否包含我们需要的query中的单、词，但仅仅这样是不够的，它无法提供文本中的字词之间的关系。


<!--
Consider the difference between these sentences:

* Sue ate the alligator.
* The alligator ate Sue.
* Sue never goes anywhere without her alligator-skin purse.

A `match` query for `sue alligator` would match all three documents, but it
doesn't tell us whether the two words form part of the same idea, or even the same
paragraph.
-->

举个例子：

* 小苏吃了鳄鱼
* 鳄鱼吃了小苏
* 小苏去哪儿都带着的鳄鱼皮钱包

用`match`查询`小苏 鳄鱼`，这三句话都会被命中，但是`tf/idf`并不会告诉我们这两个词出现在同一句话里面还是在同一个段落中（仅仅提供这两个词在这段文本中的出现频率）


<!--
Understanding how words relate to each other is a complicated problem, and
we can't solve it by just using another type of query,
but we can at least find words that appear to be related because they appear
near each other or even right next to each other.

Each document may be much longer than the examples we have presented: `Sue`
and `alligator` may be separated by paragraphs of other text. Perhaps we still
want to return these documents in which the words are widely separated, but we
want to give documents in which the words are close together a higher relevance
score.

This is the province of _phrase matching_, or _proximity matching_.

-->

理解文本中词语之间的关系是一个很复杂的问题，而且这个问题通过更换query的表达方式是无法解决的。但是我们可以知道两个词语在文本中的距离远近，甚至是否相邻，这个信息似乎上能一定程度的表达这两个词比较相关。

一般的文本可能比我们举的例子长很多，正如我们提到的：`小苏`跟`鳄鱼`这两个词可能分布在文本的不同段落中。我们还是期望能找到这两个词分布均匀的文档，但是我们把这两个词距离比较近的文档赋予更好的相关性权重。

这就是段落匹配（_phrase matching_）或者模糊匹配（_proximity matching_）所做的事情。

<!--
[TIP]
==================================================

In this chapter, we are using the same example documents that we used for
the <<match-test-data,`match` query>>.

==================================================
-->


【**提示** 】

这一章，我们会用之之前在< match-test-data, `match` query >中使用的文档做例子。

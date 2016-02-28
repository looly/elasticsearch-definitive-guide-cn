<!--秀川译-->
## 多字段搜索

只有一个简单的`match`子句的查询是很少的。我们经常需要在一个或者多个字段中查询相同的或者不同的查询字符串，意味着我们需要能够组合多个查询子句以及使他们的相关性得分有意义。

或许我们在寻找列夫·托尔斯泰写的一本叫《战争与和平》的书。或许我们在Elasticsearch的文档中查找`minimum should match`，它可能在标题中，或者在一页的正文中。或许我们查找名为John，姓为Smith的人。

在这一章节，我们会介绍用于构建多个查询子句搜索的可能的工具，以及怎么样选择解决方案来应用到你特殊的场景。

<!--
[[multi-field-search]]
== Multifield Search

Queries are seldom simple one-clause `match` queries. ((("multifield search"))) We frequently need to
search for the same or different query strings in one or more fields, which
means that we need to be able to combine multiple query clauses and their
relevance scores in a way that makes sense.

Perhaps we're looking for a book called _War and Peace_ by an author called
Leo Tolstoy. Perhaps we're searching the Elasticsearch documentation
for ``minimum should match,'' which might be in the title or the body of a
page. Or perhaps we're searching for users with first name John and last
name Smith.

In this chapter, we present the available tools for constructing multiclause
searches and how to figure out which solution you should apply to your
particular use case.
-->

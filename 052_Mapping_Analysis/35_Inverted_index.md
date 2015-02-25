## 倒排索引(Inverted index)

Elasticsearch使用一种名为_倒排索引_的结构以进行非常快速的全文文本查询。
任意文档中出现的独特单词组成一个列表，进而形成了倒排索引。
对于每一个单词而言，列表记录了它出现在哪些文档之中。

例如，我们有两个文档，每个文档的`content`字段都包含有以下内容：

1. ``The quick brown fox jumped over the lazy dog``
2. ``Quick brown foxes leap over lazy dogs in summer``

为了创建倒排索引，我们首先分割每个文档的`content`字段为独立的单词(我们称之为_terms_或_tokens_)，创建一个由所有唯一单词(unique terms)排序组成的清单。
清单中包含了每个单词(term)在文档中出现情况的分布。结果类似于这样：

    Term      Doc_1  Doc_2
    -------------------------
    Quick   |       |  X
    The     |   X   |
    brown   |   X   |  X
    dog     |   X   |
    dogs    |       |  X
    fox     |   X   |
    foxes   |       |  X
    in      |       |  X
    jumped  |   X   |
    lazy    |   X   |  X
    leap    |       |  X
    over    |   X   |  X
    quick   |   X   |
    summer  |       |  X
    the     |   X   |
    ------------------------

现在，如果我们查询`"quick brown"`，我们只需要找每个单词(term)所出现过的文档：

    Term      Doc_1  Doc_2
    -------------------------
    brown   |   X   |  X
    quick   |   X   |
    ------------------------
    Total   |   2   |  1

两个文档都匹配，但是第一个文档更适合。
如果我们使用一种简单的_相似度算法_(仅计算单词匹配的数量决定结果)，
那我们可以说第一个文档比第二个文档更匹配我们的查询(_相关性更强_)。

但是仍然有一些问题在这个倒排索引中：

1. `"Quick"`和`"quick"`被视为不同的单词，而用户认为它们其实是相同的。

2. `"fox"`和`"foxes"`是非常相似的，正如`"dog"`和`"dogs"`一样 - 它们的词根是一样的

3. `"jumped"`和`"leap"`，虽然词根不同，但是它们有相似的含义 - 它们是同义词

在以上的索引中，`"+Quick +fox"`不会匹配任何文档。(记住，单词前的`+`意味着必须存在)。
只有单词`"Quick"`和单词`"fox"`在同一个文档中才能满足这个查询，
但是第一个文档包含`"quick fox"`而第二个文档包含`"Quick foxes"`。

用户会期待两个文档都匹配此次查询，这也是很合理的。我们可以做得更好。

如果我们将单词(terms)标准化为一个标准格式，那我们就能在文档中找寻和用户查询的单词是类似含义，但不是完全一致的单词。例如：

1. `"Quick"`能被转换为小写形式`"quick"`.

2. `"foxes"`能_提取词干(stemmed)_(缩减为词根形式)成为`"fox"`。相似的还有`"dogs"`能提取词干为`"dog"`.

3. `"jumped"`及`"leap"`是同义词，能被索引为单一单词`"jump"`。

现在这个索引看起来类似于这样：

    Term      Doc_1  Doc_2
    -------------------------
    brown   |   X   |  X
    dog     |   X   |  X
    fox     |   X   |  X
    in      |       |  X
    jump    |   X   |  X
    lazy    |   X   |  X
    over    |   X   |  X
    quick   |   X   |  X
    summer  |       |  X
    the     |   X   |  X
    ------------------------

但是这还不够。我们的查询`"+Quick +fox"`*仍然*失败，因为在索引中没有单词`"Quick"`。
然而，如果我们在`content`字段的查询字串中应用相同的标准化规则，也就是`"+quick +fox"`，
就将会匹配两个文档！

重要：这是非常重要的。你仅能在索引中找到已存在的单词。所以，*索引文本及查询字串必须采用相同的标准化方式*。

标记(tokenization)及标准化(normalization)的过程被称之为_分解(analysis)_，在下一节我们将深入探讨。
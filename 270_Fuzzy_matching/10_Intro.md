[[模糊-匹配]]
== 打字错误 和 拼写错误

我们希望在结构化数据上的查询（如日期和价格）仅返回精确匹配的文档. 
((("typoes and misspellings", "fuzzy matching")))((("fuzzy matching"))) 然而, 好的全文检索不应该有同样的限制. 相反, 我们能拓宽网络以包含那些 _可能的_匹配, 并且利用相关性分数把更好的匹配结果放在结果集的前面.

事实上, 仅能精确匹配的全文检索 ((("full text search", "fuzzy matching")))可能会让你的用户感到失望. 难道你不希望一个对 ``quick brown fox'' 的检索能匹配包含
``fast brown foxes,'' 的文档，对 ``Johnny Walker'' 的检索能匹配包含
``Johnnie Walker,'' 的文档 或  ``Arnold Shcwarzenneger'' 能匹配 ``Arnold
Schwarzenegger''吗?

如果文档中存在 _确切_ 包含于用户查询的内容,它们应该出现在结果集的前面, 但更弱的匹配可能在下面的列表中.  
如果没有精确匹配的文档, 至少我们应该为用户显示可能的匹配结果; 它们甚至有可能就是用户原来想要的!

我们已经在 <<token-normalization>> 看过 diacritic-free 匹配,
在 <<stemming>> 看过 词干提取, 在 <<synonyms>> 看过 同义词, 但是所有的这些方法都预先假定了单词是正确拼写的, 或者每个词只有一种拼写方式.

模糊匹配允许 查询-时 匹配拼写错误的单词, 音标表征过滤器能在索引时用于 _发音-相似_ 的匹配.


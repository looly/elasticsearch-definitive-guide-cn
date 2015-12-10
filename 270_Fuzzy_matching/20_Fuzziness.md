[[模糊]]
=== 模糊

_模糊匹配_ 视两个单词 ``模糊'' 相似,正好像它们是同一个词.
((("typoes and misspellings", "fuzziness, defining"))) 首先, 我们需要通过_fuzziness_ 来定义什么是((("fuzziness"))).

1965年, Vladimir Levenshtein 开发了
http://en.wikipedia.org/wiki/Levenshtein_distance[Levenshtein distance(Levenshtein距离)], 用来度量把一个单词转换为另一个单词需要的单字符编辑次数 ((("Levenshtein distance"))).
他提出了3种单字符编辑:

* _替换_ 一个字符到另一个字符: _f_ox -> _b_ox

* _插入_ 一个新字符: sic -> sic_k_

* _删除_ 一个字符:: b_l_ack -> back

http://en.wikipedia.org/wiki/Frederick_J._Damerau[Frederick Damerau]
稍后扩展了这些操作并包含了1个新的 ((("Damerau, Frederick J."))):

* _换位_ 调整字符: _st_ar -> _ts_ar

例如,把 `bieber` 转换为 `beaver` 需要以下几步:

1. 用 `v` 替换掉 `b`: bie_b_er -> bie_v_er
2. 用 `a` 替换掉 `i`: b_i_ever -> b_a_ever
3. 换位 `a` 和 `e` :  b_ae_ver -> b_ea_ver

以上的3步代表了3个
http://bit.ly/1ymgZPB[Damerau-Levenshtein edit distance(Damerau-Levenshtein编辑距离)].

显然, `bieber` 距 `beaver`&#x2014很远;远得无法被认为是一个简单的拼写错误.
Damerau发现 80% 的人类拼写错误的编辑距离都是1. 换句话说, 80% 的拼写错误都可以通过 _单次编辑_ 
修改为原始的字符串.

通过指定 `fuzziness` 参数为 2,Elasticsearch 支持最大的编辑距离.

当然, 一个字符串的单次编辑次数依赖于它的长度.  对 `hat` 进行两次编辑可以得到 `mad`,
所以允许对长度为3的字符串进行两次修改就太过了. `fuzziness`
参数可以被设置成 `AUTO`, 结果会在下面的最大编辑距离中:

* `0` 1或2个字符的字符串
* `1` 3、4或5个字符的字符串
* `2` 多于5个字符的字符串

当然, 你可能发现编辑距离为`2` 仍然是太过了, 返回的结果好像并没有什么关联. 
把 `fuzziness` 设置为 `1` ,你可能会获得更好的结果和性能.

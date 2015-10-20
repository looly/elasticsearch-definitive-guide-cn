[[match-multi-word]]
=== Multiword Queries
##多词查询

如果一次只能查询一个关键词，全文检索将会很不方便。幸运的是，用``match``查询进行多词查询也很简单：

	GET /my_index/my_type/_search
	{
	    "query": {
	        "match": {
	            "title": "BROWN DOG!"
	        }
	    }
	}

上面这个查询返回以下结果集：

	{
	  "hits": [
	     {
	        "_id":      "4",
	        "_score":   0.73185337, <1>
	        "_source": {
	           "title": "Brown fox brown dog"
	        }
	     },
	     {
	        "_id":      "2",
	        "_score":   0.47486103, <2>
	        "_source": {
	           "title": "The quick brown fox jumps over the lazy dog"
	        }
	     },
	     {
	        "_id":      "3",
	        "_score":   0.47486103, <2>
	        "_source": {
	           "title": "The quick brown fox jumps over the quick dog"
	        }
	     },
	     {
	        "_id":      "1",
	        "_score":   0.11914785, <3>
	        "_source": {
	           "title": "The quick brown fox"
	        }
	     }
	  ]
	}

	<1> 文档4的相关度最高，因为包含两个``"brown"``和一个``"dog"``。

	<2> 文档2和3都包含一个``"brown"``和一个``"dog"``，且``'title'``字段长度相同，所以相关度相等。

	<3> 文档1只包含一个``"brown"``，不包含``"dog"``，所以相关度最低。

因为``match``查询需要查询两个关键词：``"brown"``和``"dog"``，在内部会执行两个``term``查询并综合二者的结果得到最终的结果。``match``的实现方式是将两个``term``查询放入一个``bool``查询，``bool``查询在之前的章节已经介绍过。

重要的一点是，``'title'``字段包含_至少一个_查询关键字的文档都被认为是符合查询条件的。匹配的单词数越多，文档的相关度越高。

[[match-improving-precision]]
==== Improving Precision

### 提高精度

匹配包含任意个数查询关键字的文档可能会得到一些看似不相关的结果，这是一种霰弹策略(shotgun approach)。然而我们可能想得到包含_所有_查询关键字的文档。换句话说，我们想得到的是匹配``'brown AND dog'``的文档，而非``'brown OR dog'``。

``match``查询接受一个``'operator'``参数，默认值为``or``。如果要求所有查询关键字都匹配，可以更改参数值为``and``：

	GET /my_index/my_type/_search
	{
	    "query": {
	        "match": {
	            "title": {      <1>
	                "query":    "BROWN DOG!",
	                "operator": "and"
	            }
	        }
	    }
	}

	<1> The structure of the ``match`` query has to change slightly in order to
    accommodate the ``'operator'`` parameter.

这个查询会排除文档1，因为文档1只包含了一个查询关键词。

[[match-precision]]
==== Controlling Precision

### 控制精度

在 _all_ 和 _any_ 之间的选择有点过于非黑即白。如果用户指定了5个查询关键字，而一个文档只包含了其中的4个？将``'operator'``设置为``'and'``会排除这个文档。

有时这的确是用户想要的结果。但在大多数全文检索的使用场景下，用户想得到相关的文档，排除那些不太可能相关的文档。换句话说，我们需要介于二者之间的选项。

The `match` query supports((("match query", "minimum_should_match parameter")))((("minimum_should_match parameter"))) the `minimum_should_match` parameter, which allows
you to specify the number of terms that must match for a document to be considered
relevant.  While you can specify an absolute number of terms, it usually makes
sense to specify a percentage instead, as you have no control over the number of words the user may enter:

[source,js]
--------------------------------------------------
GET /my_index/my_type/_search
{
  "query": {
    "match": {
      "title": {
        "query":                "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
--------------------------------------------------
// SENSE: 100_Full_Text_Search/05_Match_query.json

When specified as a percentage, `minimum_should_match` does the right thing:
in the preceding example with three terms, `75%` would be rounded down to `66.6%`,
or two out of the three terms. No matter what you set it to, at least one term
must match for a document to be considered a match.

[NOTE]
====
The `minimum_should_match` parameter is flexible, and different rules can
be applied depending on the number of terms the user enters.  For the full
documentation see the
http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html#query-dsl-minimum-should-match
====

To fully understand how the `match` query handles multiword queries, we need
to look at how to combine multiple queries with the `bool` query.

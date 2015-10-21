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

	<1> 文档4的相关度最高，因为包含两个"brown"和一个"dog"。

	<2> 文档2和3都包含一个"brown"和一个"dog"，且'title'字段长度相同，所以相关度相等。

	<3> 文档1只包含一个"brown"，不包含"dog"，所以相关度最低。

因为``match``查询需要查询两个关键词：``"brown"``和``"dog"``，在内部会执行两个``term``查询并综合二者的结果得到最终的结果。``match``的实现方式是将两个``term``查询放入一个``bool``查询，``bool``查询在之前的章节已经介绍过。

重要的一点是，``'title'``字段包含_至少一个_查询关键字的文档都被认为是符合查询条件的。匹配的单词数越多，文档的相关度越高。

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

	<1> 为了加入``'operator'``参数，``match``查询的结构有一些不同。
	
这个查询会排除文档1，因为文档1只包含了一个查询关键词。

### 控制精度

在 _all_ 和 _any_ 之间的选择有点过于非黑即白。如果用户指定了5个查询关键字，而一个文档只包含了其中的4个？将``'operator'``设置为``'and'``会排除这个文档。

有时这的确是用户想要的结果。但在大多数全文检索的使用场景下，用户想得到相关的文档，排除那些不太可能相关的文档。换句话说，我们需要介于二者之间的选项。

``match``查询有``'minimum_should_match'``参数，参数值表示被视为_相关_的文档必须匹配的关键词个数。参数值可以设为整数，也可以设置为百分数。因为不能提前确定用户输入的查询关键词个数，使用百分数也很合理。

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

当``'minimum_should_match'``被设置为百分数时，查询进行如下：在上面的例子里，``'75%'``会被下舍为``'66.6%'``，也就是2个关键词。不论参数值为多少，进入结果集的文档至少应匹配一个关键词。

####[提示]

``'minimum_should_match'``参数很灵活，根据用户输入的关键词个数，可以采用不同的匹配规则。更详细的内容可以查看[文档](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html#query-dsl-minimum-should-match)。

要全面理解``match``查询是怎样处理多词查询，我们需要了解怎样用``bool``查询合并多个查询。

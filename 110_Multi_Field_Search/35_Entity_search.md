#### 跨字段实体搜索(Cross-fields Entity Search)

现在让我们看看一个常见的模式：跨字段实体搜索。类似person，product或者address这样的实体，它们的信息会分散到多个字段中。我们或许有一个person实体被索引如下：

```Javascript
{
    "firstname":  "Peter",
    "lastname":   "Smith"
}
```

而address实体则是像下面这样：

```Javascript
{
    "street":   "5 Poland Street",
    "city":     "London",
    "country":  "United Kingdom",
    "postcode": "W1V 3DG"
}
```

这个例子也许很像在[多查询字符串](../110_Multi_Field_Search/05_Multiple_query_strings.md)中描述的，但是有一个显著的区别。在多查询字符串中，我们对每个字段都使用了不同的查询字符串。在这个例子中，我们希望使用一个查询字符串来搜索多个字段。

用户也许会搜索名为"Peter Smith"的人，或者名为"Poland Street W1V"的地址。每个查询的单词都出现在不同的字段中，因此使用dis_max/best_fields查询来搜索单个最佳匹配字段显然是不对的。

#### 一个简单的方法

实际上，我们想要依次查询每个字段然后将每个匹配字段的分值进行累加，这听起来很像bool查询能够胜任的工作：

```Javascript
{
  "query": {
    "bool": {
      "should": [
        { "match": { "street":    "Poland Street W1V" }},
        { "match": { "city":      "Poland Street W1V" }},
        { "match": { "country":   "Poland Street W1V" }},
        { "match": { "postcode":  "Poland Street W1V" }}
      ]
    }
  }
}
```

对每个字段重复查询字符串很快就会显得冗长。我们可以使用multi_match查询进行替代，然后将type设置为most_fields来让它将所有匹配字段的分值合并：

```Javascript
{
  "query": {
    "multi_match": {
      "query":       "Poland Street W1V",
      "type":        "most_fields",
      "fields":      [ "street", "city", "country", "postcode" ]
    }
  }
}
```

#### 使用most_fields存在的问题

使用most_fields方法执行实体查询有一些不那么明显的问题：

* 它被设计用来找到匹配任意单词的多数字段，而不是找到跨越所有字段的最匹配的单词。
* 它不能使用operator或者minimum_should_match参数来减少低相关度结果带来的长尾效应。
* 每个字段的词条频度是不同的，会互相干扰最终得到较差的排序结果。



<!--
=== Cross-fields Entity Search

Now we come to a common pattern: cross-fields entity search. ((("cross-fields entity search")))((("multifield search", "cross-fields entity search"))) With entities
like `person`, `product`, or `address`, the identifying information is spread
across several fields.  We may have a `person` indexed as follows:

[source,js]
--------------------------------------------------
{
    "firstname":  "Peter",
    "lastname":   "Smith"
}
--------------------------------------------------

Or an address like this:

[source,js]
--------------------------------------------------
{
    "street":   "5 Poland Street",
    "city":     "London",
    "country":  "United Kingdom",
    "postcode": "W1V 3DG"
}
--------------------------------------------------

This sounds a lot like the example we described in <<multi-query-strings>>,
but there is a big difference between these two scenarios.  In
<<multi-query-strings>>, we used a separate query string for each field. In
this scenario, we want to search across multiple fields with a _single_ query
string.

Our user might search for the person ``Peter Smith'' or for the address
``Poland Street W1V.'' Each of those words appears in a different field, so
using a `dis_max` / `best_fields` query to find the _single_ best-matching
field is clearly the wrong approach.

==== A Naive Approach

Really, we want to query each field in turn and add up the scores of every
field that matches, which sounds like a job for the `bool` query:

[source,js]
--------------------------------------------------
{
  "query": {
    "bool": {
      "should": [
        { "match": { "street":    "Poland Street W1V" }},
        { "match": { "city":      "Poland Street W1V" }},
        { "match": { "country":   "Poland Street W1V" }},
        { "match": { "postcode":  "Poland Street W1V" }}
      ]
    }
  }
}
--------------------------------------------------

Repeating the query string for every field soon becomes tedious. We can use
the `multi_match` query instead, ((("most fields queries", "problems for entity search")))((("multi_match queries", "most_fields type")))and set the `type` to `most_fields` to tell it to
combine the scores of all matching fields:

[source,js]
--------------------------------------------------
{
  "query": {
    "multi_match": {
      "query":       "Poland Street W1V",
      "type":        "most_fields",
      "fields":      [ "street", "city", "country", "postcode" ]
    }
  }
}
--------------------------------------------------

==== Problems with the most_fields Approach

The `most_fields` approach to entity search has some problems that are not
immediately obvious:

* It is designed to find the most fields matching _any_ words, rather than to
  find the most matching words _across all fields_.

* It can't use the `operator` or `minimum_should_match` parameters
  to reduce the long tail of less-relevant results.

* Term frequencies are different in each field and could interfere with each
  other to produce badly ordered results.
  -->

## 扁平化你的数据

Elasticsearch 鼓励你在创建索引的时候就 [扁平化（denormalizing）](http://en.wikipedia.org/wiki/Denormalization) 你的数据，这样做可以获取最好的搜索性能。在每一篇文档里面冗余一些数据可以避免join操作。

举个例子，如果我们想通过 `user` 来查找某一篇 `blog`，那么就把 `user` 的姓名包含在 `blog` 这个 document 里面，就像这样：

``` json

PUT /my_index/user/1
{
  "name":     "John Smith",
  "email":    "john@smith.com",
  "dob":      "1970/10/24"
}

PUT /my_index/blogpost/2
{
  "title":    "Relationships",
  "body":     "It's complicated...",
  "user":     {
    "id":       1,
    "name":     "John Smith"  (1)
  }
}

```

(1) `user` 中的一部分信息已经被包含到 `blogpost`  里面。


现在，我们只要通过一次查询，就能找到和作者 `John` 有关系的所有博客：


```

GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title":     "relationships" }},
        { "match": { "user.name": "John"          }}
      ]
    }
  }
}

```

扁平化数据的好处就是一个字，快。因为每一篇文档都已经包含了所有需要被查询的信息，所以就没有必要去做消耗很大的join操作了。

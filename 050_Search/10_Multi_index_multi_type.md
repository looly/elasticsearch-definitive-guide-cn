## 多索引和多类别

你注意到空搜索的结果中不同类型的文档——`user`和`tweet`——来自于不同的索引——`us`和`gb`。

通过限制我们的搜索到不同的索引或类型，我们可以在集群中跨**所有**文档搜索。Elasticsearch转发搜索请求到集群中平行的主分片或每个分片的复制分片上，收集结果然后选择顶部十个返回给我们。

通常，当然，你可能想搜索一个或几个自定的索引或类型，我们能通过定义URL中的索引或类型达到这个目的，像这样：

```javascript
`/_search`::

    search all types in all indices

`/gb/_search`::

    search all types in the `gb` index

`/gb,us/_search`::

    search all types in the `gb` and `us` indices

`/g*,u*/_search`::

    search all types in any indices beginning with `g` or beginning with `u`

`/gb/user/_search`::

    search type `user` in the `gb` index

`/gb,us/user,tweet/_search`::

    search types `user` and `tweet` in the `gb` and `us` indices

`/_all/user,tweet/_search`::

    search types `user` and `tweet` in all indices
```

当你搜索包含单一索引时，Elasticsearch转发搜索请求到这个索引的主分片或每个分片的复制分片上，然后聚集每个分片的结果。搜索包含多个索引也是同样的方式——只不过或有更多的分片被关联。

> ## 重要

> 搜索一个索引有5个主分片和5个索引各有一个分片**事实上是一样的**。

接下来，你将看到这些简单的情况如何灵活的扩展以适应你需求的变更。

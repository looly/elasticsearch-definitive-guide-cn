## 应用级别的Join操作

我们可以在应用这一层面（部分的）模仿实现关系数据库中的join操作。例如，我们要给 `users`  以及每个 `user` 所对应的若干篇 `blog` 建立索引。在这充满关系的世界中，我们可以做一些类似于这样的事情：

```
PUT /my_index/user/1     (1)
{
  "name":     "John Smith",
  "email":    "john@smith.com",
  "dob":      "1970/10/24"
}

PUT /my_index/blogpost/2 (1)
{
  "title":    "Relationships",
  "body":     "It's complicated...",
  "user":     1          (2)
}
```
(1) 每一个 document 中 `index`，`type`，和 `id`  共同组成了主键。

(2) `blogpost` 通过包含 `user` 的 `id` 来关联 `user`，而这里不需要指定 `user` 的 `index` 和 `type` 是因为在我们的应用中它们是被硬编码的（这里的硬编码的意思应该是说，在 `blogpost` document中引用了 `user` ，那么es就会在相同的index下查找 `user` type，并且id为1的document，所以不需要指定 `index` 和 `type`）。

通过查询 `user` 的ID为1将很容易找到相应的 `blog`：

```
GET /my_index/blogpost/_search
{
  "query": {
    "filtered": {
      "filter": {
        "term": { "user": 1 }
      }
    }
  }
}
```
想通过博客作者的名字 `John` 来找到相关的博客，我们需要执行2个查询语句：
第一，我们需要先找到所有叫 `John` 的博客作者，从而获得它们的 ID列表，
第二，将获取到的ID列表作为查询条件来执行类似于上面的查询语句：

```
GET /my_index/user/_search
{
  "query": {
    "match": {
      "name": "John"
    }
  }
}

GET /my_index/blogpost/_search
{
  "query": {
    "filtered": {
      "filter": {
        "terms": { "user": [1] }  (1)
      }
    }
  }
}
```
(1) 其中 `terms` 的值被设置成从第一个查询中得到的ID列表。

在应用级别模仿join操作的最大好处是数据是立体的（normalized），如果想改变  `user` 的姓名，那么只要在 `user` 这个 document 上改就可以了。而缺点是你必须在查询期间运行额外的 query 来实现 join 的操作。

在这个例子当中，只有一个 `user` 符合我们的第一个查询条件，但在真实的世界中，很可能会出现数百万人的名字叫 `John`，将这么多的ID塞到第二个查询中，将会让这个查询语句变得非常庞大，并且这个查询会执行数百万次 `term` 的查找。

这种模仿join操作的方法适合于前置查询结果集（在该例子中指代 `user`）比较小，并且最好是不经常变化的，此时我们在应用中可以去缓存这部分数据，避免频繁的执行第一个查询。






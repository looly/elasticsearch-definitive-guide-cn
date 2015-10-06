#近实时搜索

因为per-segment search 机制，索引和搜索一个文档之间是有延迟的。新的文档会在几分钟内可以搜索，但是这依然不够快。

磁盘是瓶颈。提交一个新的段到磁盘需要 fsync操作，确保段被物理地写入磁盘，即时电源失效也不会丢失数据。但是fsync是昂贵的，它不能在每个文档被索引的时就触发。

所以需要一种更轻量级的方式使新的文档可以被搜索，这意味这移除fsync

位于Elasticsearch和磁盘间的是文件系统缓存。如前所说，在内存索引缓存中的文档被写入新的段，但是新的段首先写入文件系统缓存，这代价很低，之后会被同步到磁盘，这个代价很大。但是一旦一个文件被缓存，它也可以被打开和读取，就像其他文件一样。

**图1：内存缓存区有新文档的Lucene索引**
![内存缓存区有新文档的Lucene索引](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1104.png)

Lucene允许新段写入打开，好让它们包括的文档可搜索，而不用执行一次全量提交。这是比提交更轻量的过程，可以经常操作，而不会影响性能。

**图2：缓存内容已经写到段中，但是还没提交**
![缓存内容已经写到段中，但是还没提交](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1105.png)

##refeash API
在Elesticsearch中，这种写入打开一个新段的轻量级过程，叫做refresh。默认情况下，每个分片每秒自动刷新一次。这就是为什么说Elasticsearch是近实时的搜索了：文档的改动不会立即被搜索，但是会在一秒内可见。

这会困扰新用户：他们索引了个文档，尝试搜索它，但是搜不到。解决办法就是执行一次人工刷新，通过API:

[source,json]
-----------------------------
POST /_refresh <1>
POST /blogs/_refresh <2>
-----------------------------
<1> Refresh all indices.
<2> Refresh just the `blogs` index.
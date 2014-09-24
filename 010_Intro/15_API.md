## 与Elasticsearch交互

如何与ELasticsearch交互取决于你是否使用Java。

### Java API

如果你使用Java，ELasticsearch提供两种内置客户端用于你的代码：

#### 节点客户端(node client)：
节点客户端以**无数据节点(none data node)**身份加入集群，换言之，它自己没有数据，但是知道什么数据位于集群的哪个节点上，能够直接转发请求到对应的节点上。

#### 传输客户端(Transport client)：
更轻量的传输客户端能够发送请求到远程集群，它自己不加入集群，只是简单转发请求给集群中的节点。

两个Java客户端都通过**9300端口**与集群交互，使用**Elasticsearch传输协议(Elasticsearch Transport Protocol)**。集群中的节点也通过**9300端口**通信。如果此端口未开放，你的节点将不能形成集群。

>**提示**

>Java客户端版本必须与集群中的节点一致，换言之，它们可能互相无法识别。

关于Java API的更多信息请查看相关章节：[Java API](http://www.elasticsearch.org/guide/)

### 通过HTTP使用附带JSON数据的RESTful API
其他所有程序语言都可以通过**9200端口**的RESTful API与Elasticsearch通信，事实上，如你所见，你甚至可以通过命令行的`curl`命令与ELasticsearch通信。

Elasticsearch官方提供了多种程序语言的客户端——Groovy，Javascript， .NET，PHP，Perl，Python，and Ruby——还有数量庞大的社区提供的客户端和集成插件，所有这些可以在这里找到：[Guide](http://www.elasticsearch.org/guide/)。

向Elasticsearch发出的请求与其它HTTP请求组成是一致的，例如统计集群中文件的数量，我们可以这样：

```bash
      <1>     <2>                     <3>    <4>
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{  <5>
    "query": {
        "match_all": {}
    }
}
'
```
--------------------------------------------------
- <1> 合适的HTTP方法或动词：`GET`, `POST`, `PUT`, `HEAD` or `DELETE`。
- <2> 任意一个节点的协议、主机名和端口。
- <3> 请求路径。
- <4> 一些可选的查询参数，例如`?pretty`返回更加美观易读的JSON。
- <5> 一个JSON编码过的请求主体（如果需要）。

Elasticsearch返回一个类似`200 OK`的HTTP状态码和JSON格式主体（除`HEAD`请求）。上面的请求会得到如下的响应主体：

```Javascript
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```

我们看不到HTTP头是因为没有让`curl`显示它们，如果想显示，使用`curl`命令后跟`-i`参数:

```Javascript
curl -i -XGET 'localhost:9200/'
```

对于本书的其余部分，我们将简写`curl`中每次重复的部分，例如主机名和端口，还有`curl`命令本身。原代码：

```Javascript
curl -XGET 'localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```
我们将简写成这样：

```Javascript
GET /_count
{
    "query": {
        "match_all": {}
    }
}
```

事实上，这与在**Sense终端**中使用的格式相同，你可以点击顶部的`View in Sense`来运行这段代码。

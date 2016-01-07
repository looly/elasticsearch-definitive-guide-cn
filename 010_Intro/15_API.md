## 与Elasticsearch交互

如何与Elasticsearch交互取决于你是否使用Java。

### Java API

Elasticsearch为Java用户提供了两种内置客户端：

#### 节点客户端(node client)：
节点客户端以无数据节点(none data node)身份加入集群，换言之，它自己不存储任何数据，但是它知道数据在集群中的具体位置，并且能够直接转发请求到对应的节点上。

#### 传输客户端(Transport client)：
这个更轻量的传输客户端能够发送请求到远程集群。它自己不加入集群，只是简单转发请求给集群中的节点。

两个Java客户端都通过9300端口与集群交互，使用Elasticsearch传输协议(Elasticsearch Transport Protocol)。集群中的节点之间也通过9300端口进行通信。如果此端口未开放，你的节点将不能组成集群。

>**TIP**

>Java客户端所在的Elasticsearch版本必须与集群中其他节点一致，否则，它们可能互相无法识别。

关于Java API的更多信息请查看相关章节：[Java API](http://www.elasticsearch.org/guide/)

### 基于HTTP协议，以JSON为数据交互格式的RESTful API
其他所有程序语言都可以使用RESTful API，通过9200端口的与Elasticsearch进行通信，你可以使用你喜欢的WEB客户端，事实上，如你所见，你甚至可以通过`curl`命令与Elasticsearch通信。

> **NOTE**

>Elasticsearch官方提供了多种程序语言的客户端——Groovy，Javascript， .NET，PHP，Perl，Python，以及 Ruby——还有很多由社区提供的客户端和插件，所有这些可以在[文档](http://www.elasticsearch.org/guide/)中找到。

向Elasticsearch发出的请求的组成部分与其它普通的HTTP请求是一样的：
```bash
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```
--------------------------------------------------
- VERB         HTTP方法：`GET`, `POST`, `PUT`, `HEAD`, `DELETE`
- PROTOCOL     http或者https协议（只有在Elasticsearch前面有https代理的时候可用）
- HOST         Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
- PORT         Elasticsearch HTTP服务所在的端口，默认为9200
- PATH         API路径（例如_count将返回集群中文档的数量），PATH可以包含多个组件，例如_cluster/stats或者_nodes/stats/jvm
- QUERY_STRING 一些可选的查询请求参数，例如`?pretty`参数将使请求返回更加美观易读的JSON数据
- BODY         一个JSON格式的请求主体（如果请求需要的话）

举例说明，为了计算集群中的文档数量，我们可以这样做：

```Javascript
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```

Elasticsearch返回一个类似`200 OK`的HTTP状态码和JSON格式的响应主体（除了`HEAD`请求）。上面的请求会得到如下的JSON格式的响应主体：

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

我们看不到HTTP头是因为我们没有让`curl`显示它们，如果要显示，使用`curl`命令后跟`-i`参数:

```Javascript
curl -i -XGET 'localhost:9200/'
```

对于本书的其余部分，我们将简写`curl`请求中重复的部分，例如主机名和端口，还有`curl`命令本身。

一个完整的请求形如：

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

事实上，在Sense控制台中也使用了与上面相同的格式。

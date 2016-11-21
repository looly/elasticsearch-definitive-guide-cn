## 安装Elasticsearch

理解Elasticsearch最好的方式是去运行它，让我们开始吧！

安装Elasticsearch唯一的要求是安装官方新版的Java，地址：[www.java.com](http://www.java.com)

你可以从 [elasticsearch.org\/download](http://www.elasticsearch.org/download/) 下载最新版本的Elasticsearch。

```bash
curl -L -O http://download.elasticsearch.org/PATH/TO/VERSION.zip <1>
unzip elasticsearch-$VERSION.zip
cd  elasticsearch-$VERSION
```

1. 从 [elasticsearch.org\/download](http://www.elasticsearch.org/download/) 获得最新可用的版本号并填入URL中

> **提示：**
> 
> 在生产环境安装时，除了以上方法，你还可以使用Debian或者RPM安装包，地址在这里：[downloads page](http://www.elasticsearch.org/downloads)，或者也可以使用官方提供的 [Puppet module](https://github.com/elasticsearch/puppet-elasticsearch) 或者
> [Chef cookbook](https://github.com/elasticsearch/cookbook-elasticsearch)。

## 安装Marvel

[Marvel](http://www.elasticsearch.com/marvel)是Elasticsearch的管理和监控工具，在开发环境下免费使用。它包含了一个叫做`Sense`的交互式控制台，使用户方便的通过浏览器直接与Elasticsearch进行交互。

Elasticsearch线上文档中的很多示例代码都附带一个`View in Sense`的链接。点击进去，就会在`Sense`控制台打开相应的实例。安装Marvel不是必须的，但是它可以通过在你本地Elasticsearch集群中运行示例代码而增加与此书的互动性。

Marvel是一个插件，可在Elasticsearch目录中运行以下命令来下载和安装：

```bash
./bin/plugin -i elasticsearch/marvel/latest
```

你可能想要禁用监控，你可以通过以下命令关闭Marvel：

```bash
echo 'marvel.agent.enabled: false' >> ./config/elasticsearch.yml
```

## 运行Elasticsearch

Elasticsearch已经准备就绪，执行以下命令可在前台启动：

```bash
./bin/elasticsearch
```

启动后，如果只有本地可以访问，尝试修改配置文件 elasticsearch.yml

中network.host\(注意配置文件格式不是以`#`开头的要空一格， `：`后要空一格\) 为`network.host: 0.0.0.0`

如果想在后台以守护进程模式运行，添加`-d`参数。

打开另一个终端进行测试：

```bash
curl 'http://localhost:9200/?pretty'
```

你能看到以下返回信息：

```javascript
{
   "status": 200,
   "name": "Shrunken Bones",
   "version": {
      "number": "1.4.0",
      "lucene_version": "4.10"
   },
   "tagline": "You Know, for Search"
}
```

这说明你的ELasticsearch集群已经启动并且正常运行，接下来我们可以开始各种实验了。

## 集群和节点

**节点\(node\)**是一个运行着的Elasticsearch实例。**集群\(cluster\)**是一组具有相同`cluster.name`的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一个节点也可以组成一个集群。

你最好找一个合适的名字来替代`cluster.name`的默认值，比如你自己的名字，这样可以防止一个新启动的节点加入到相同网络中的另一个同名的集群中。

你可以通过修改`config/`目录下的`elasticsearch.yml`文件，然后重启ELasticsearch来做到这一点。当Elasticsearch在前台运行，可以使用`Ctrl-C`快捷键终止，或者你可以调用`shutdown` API来关闭：

```bash
curl -XPOST 'http://localhost:9200/_shutdown'
```

## 查看Marvel和Sense

如果你安装了Marvel（作为管理和监控的工具），就可以在浏览器里通过以下地址访问它：

[http:\/\/localhost:9200\/\_plugin\/marvel\/](http://localhost:9200/_plugin/marvel/)

你可以在Marvel中通过点击`dashboards`，在下拉菜单中访问**Sense**开发者控制台，或者直接访问以下地址：

[http:\/\/localhost:9200\/\_plugin\/marvel\/sense\/](http://localhost:9200/_plugin/marvel/sense/)


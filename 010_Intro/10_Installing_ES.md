## 安装Elasticsearch

理解Elasticsearch最好的方式是去运行它，让我们开始吧！

安装Elasticsearch唯一的要求是官方新版的Java，地址：[www.java.com](http://www.java.com)

你可以从 [elasticsearch.org/download](http://www.elasticsearch.org/download/) 下载最新版本的Elasticsearch。

```bash
curl -L -O http://download.elasticsearch.org/PATH/TO/VERSION.zip <1>
unzip elasticsearch-$VERSION.zip
cd  elasticsearch-$VERSION
```
1. 从 [elasticsearch.org/download](http://www.elasticsearch.org/download/) 获得最新可用的版本号并填入URL中

>**提示：**

>在生产环境安装时，除了以上方法，你还可以使用Debian或者RPM安装包，地址在这里：[downloads page](http://www.elasticsearch.org/downloads)，或者也可以使用官方提供的 [Puppet module](https://github.com/elasticsearch/puppet-elasticsearch) 或者
[Chef cookbook](https://github.com/elasticsearch/cookbook-elasticsearch)。

## 安装Marvel

[Marvel](http://www.elasticsearch.com/marvel)是Elasticsearch的管理和监控工具，对于开发使用免费的。它配备了一个叫做`Sense`的交互式控制台，方便通过浏览器直接与Elasticsearch交互。

很多代码样例包含叫做`View in Sense` 的链接，点击会后会在Sense控制台上打开代码样例。

安装Marvel不是必须，但是它可以通过在你本地Elasticsearch集群中运行样例代码而增加与此书的互动性。

Marvel是一个插件，在Elasticsearch目录中运行以下代码来下载和安装：

```bash
./bin/plugin -i elasticsearch/marvel/latest
```

你可能想要禁用监控，你可以通过以下代码关闭Marvel：

```bash
echo 'marvel.agent.enabled: false' >> ./config/elasticsearch.yml
```

## 运行Elasticsearch

Elasticsearch已经准备就绪，执行以下命令在前台启动：

```bash
./bin/elasticsearch
```
如果想在后台以守护进程模式运行，添加`-d`参数。

打开另一个终端运行测试：

```bash
curl 'http://localhost:9200/?pretty'
```

你能看到以下信息：

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
这说明你的ELasticsearch集群已经启动并运行，接下来你可以开始尝试各种功能了。

## 集群和节点

**节点(node)**是你运行的Elasticsearch实例。一个**集群(cluster)**是一组具有相同`cluster.name`的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一个节点也可以组成一个集群。

你需要修改`cluster.name`默认值为适合你的值，用于停止节点加入到同网络相同名字的集群中。

你可以修改`config/`目录下的`elasticsearch.yml`文件，然后重启ELasticsearch。当Elasticsearch前台运行，使用`Ctrl-C`快捷键终止，否则你可以调用`shutdown` API关闭：

```bash
curl -XPOST 'http://localhost:9200/_shutdown'
```

## 查看Marvel和Sense

如果你安装了Marvel管理和监控工具，可以通过在浏览器里通过以下地址访问：

[http://localhost:9200/_plugin/marvel/](http://localhost:9200/_plugin/marvel/)

你可以在Marvel中的`Marvel dashboards`点击下拉菜单或者访问以下地址访问**Sense**开发者控制台：

[http://localhost:9200/_plugin/marvel/sense/](http://localhost:9200/_plugin/marvel/sense/)







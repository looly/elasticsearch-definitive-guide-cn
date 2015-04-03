## 开始第一步
我们现在开始进行一个简单教程，它涵盖了一些基本的概念介绍，比如**索引(indexing)**、**搜索(search)**以及**聚合(aggregations)**。通过这个教程，我们可以让你对Elasticsearch能做的事以及其易用程度有一个大致的感觉。

我们接下来将陆续介绍一些术语和基本的概念，但就算你没有马上完全理解也没有关系。我们将在本书的各个章节中更加深入的探讨这些内容。

所以，坐下来，开始以旋风般的速度来感受Elasticsearch的能力吧！

## 让我们建立一个员工目录
假设我们刚好在**Megacorp**工作，这时人力资源部门出于某种目的需要让我们创建一个员工目录，这个目录用于促进人文关怀和用于实时协同工作，所以它有以下不同的需求：

* 数据能够包含多个值的标签、数字和纯文本。
* 检索任何员工的所有信息。
* 支持结构化搜索，例如查找30岁以上的员工。
* 支持简单的全文搜索和更复杂的**短语(phrase)**搜索
* 高亮搜索结果中的关键字
* 能够利用图表管理分析这些数据

## 索引员工文档
首要的工作是存储员工数据。这将需要一个“员工文档”的表单，每个文档代表一个员工。在Elasticsearch中存储数据的行为叫做**索引(indexing)**，不过在索引之前，我们需要决定数据存储在哪里。

在Elasticsearch中，文档属于一种**类型(type)**,然后这些类型存在于**索引(index)**中，你可以画一些简单的对比图来类比传统关系型数据库中的一些概念：
```
关系数据库(Relational DB) -> 库(Databases) -> 表(Tables) -> 行(Rows)       -> 列(Columns)
Elasticsearch           -> 索引(Indices) -> 类型(Types) -> 文档(Documents) -> 字段(Fields)
```

Elasticsearch集群可以包含多个**索引(indices)**（数据库），这些库可以包含多个**类型(types)**（表），这些类型包含多个**文档(documents)**（行），然后每个文档包含多个**字段(Fields)**（列）。

**************************************************
>### 名词索引、动词索引和反向索引的区分

你可能已经注意到**索引(index)**这个词在Elasticsearch中有着不同的含义，所以在此有必要做一下澄清：

>#### 索引（名词）

如上文所述，一个**索引(index)**像是传统关系数据库中的**数据库**，它是相关文档存储的地方，index的复数是**indices **或**indexes**。

>#### 索引（动词）

**“索引一个文档”**表示存储一个文档在**索引（名词）**里，以便它可以被检索或者查询。这很像SQL中的`INSERT`关键字，差别是，如果文档已经存在，新的文档将覆盖旧的文档。

>#### 反向索引

传统数据库为特定列增加一个**索引(index)**，例如多路搜索树(B-Tree)索引来加速检索。Elasticsearch和Lucene使用一种叫做**反向索引(inverted index)**的结构来实现相同目的。

通常文档中的所有字段会被**索引**（拥有反向索引），因此他们可以被搜索。如果一个字段没有反向索引不能被搜索。

我们将会在“反向索引”章节更详细的讨论。

**************************************************

所以为了创建员工目录，我们将进行如下操作：

* 为每个员工的**文档(document)**建立索引，这个文档包含了每个员工的所有信息。
* 每个文档属于叫做`employee`**类型(type)**。
* 这个类型存在于叫做`megacorp`的**索引(index)**中。
* 这个索引存储在Elasticsearch集群中。

实践中，这些都是很容易的（尽管看起来许多步骤）。我们能通过一个命令执行所有的操作：

```Javascript
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

注意到路径`/megacorp/employee/1`包含三部分信息：

| 名字       | 说明        |
| ---------- | ----------- |
|**megacorp**|索引名       |
|**employee**|类型名       |
|**1**       |这个职员的ID |

请求体——JSON文档，包含了这个职员的所有信息。他的名字叫“John Smith”，25岁，喜欢攀岩。

很简单吧，操作前不需要任何管理操作，比如创建索引或者定义每个字段的数据类型。我们能够直接索引文档，Elasticsearch附带所有的缺省设置，所有管理操作已经在后台帮你执行。

在继续之前，让我们在目录中加入更多员工信息：

```Javascript
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```


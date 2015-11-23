## 复合核心字段类型

除了之前提到的简单的标量类型，JSON还有`null`值，数组和对象，所有这些Elasticsearch都支持：

### 多值字段

我们想让`tag`字段包含多个字段，这非常有可能发生。我们可以索引一个标签数组来代替单一字符串：

```javascript
{ "tag": [ "search", "nosql" ]}
```

对于数组不需要特殊的映射。任何一个字段可以包含零个、一个或多个值，同样对于全文字段将被分析并产生多个词。

言外之意，这意味着**数组中所有值必须为同一类型**。你不能把日期和字符窜混合。如果你创建一个新字段，这个字段索引了一个数组，Elasticsearch将使用第一个值的类型来确定这个新字段的类型。

当你从Elasticsearch中取回一个文档，任何一个数组的顺序和你索引它们的顺序一致。你取回的`_source`字段的顺序同样与索引它们的顺序相同。

然而，数组是做为多值字段被**索引**的，它们没有顺序。在搜索阶段你不能指定“第一个值”或者“最后一个值”。倒不如把数组当作一个**值集合(gag of values)**

==== Empty fields
### 空字段

当然数组可以是空的。这等价于有零个值。事实上，Lucene没法存放`null`值，所以一个`null`值的字段被认为是空字段。

这四个字段将被识别为空字段而不被索引：

```javascript
"empty_string":             "",
"null_value":               null,
"empty_array":              [],
"array_with_null_value":    [ null ]
```

### 多层对象

我们需要讨论的最后一个自然JSON数据类型是**对象(object)**——在其它语言中叫做hashed、hashmaps、dictionaries 或者 associative arrays.

**内部对象(inner objects)**经常用于嵌入一个实体或对象里的另一个地方。例如，做在`tweet`文档中`user_name`和`user_id`的替代，我们可以这样写：

```javascript
{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}
```


### 内部对象的映射

Elasticsearch 会动态的检测新对象的字段，并且映射它们为 `object` 类型，将每个字段加到 `properties` 字段下

```json
{
  "gb": {
    "tweet": { <1>
      "properties": {
        "tweet":            { "type": "string" },
        "user": { <2>
          "type":             "object",
          "properties": {
            "id":           { "type": "string" },
            "gender":       { "type": "string" },
            "age":          { "type": "long"   },
            "name":   { <2>
              "type":         "object",
              "properties": {
                "full":     { "type": "string" },
                "first":    { "type": "string" },
                "last":     { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
```

<1> 根对象.
<2> 内部对象.


对`user`和`name`字段的映射与`tweet`类型自己很相似。事实上，`type`映射只是`object`映射的一种特殊类型，我们将 `object` 称为_根对象_。它与其他对象一模一样，除非它有一些特殊的顶层字段，比如 `_source`, `_all` 等等。

### 内部对象是怎样被索引的

Lucene 并不了解内部对象。 一个 Lucene 文件包含一个键-值对应的扁平表单。 为了让 Elasticsearch 可以有效的索引内部对象，将文件转换为以下格式：

```javascript
{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
```

_内部栏位_可被归类至name，例如`"first"`。 为了区别两个拥有相同名字的栏位，我们可以使用完整_路径_，例如`"user.name.first"` 或甚至`类型`名称加上路径：`"tweet.user.name.first"`。

注意： 在以上扁平化文件中，并没有栏位叫作`user`也没有栏位叫作`user.name`。 Lucene 只索引阶层或简单的值，而不会索引复杂的资料结构。

## 对象-阵列
### 内部对象的阵列

最後，一个包含内部对象的阵列如何索引。 我们有个阵列如下所示：

```json
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
```


此文件会如我们以上所说的被扁平化，但其结果会像如此：

```json
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
```


`{age: 35}`与`{name: Mary White}`之间的关联会消失，因每个多值的栏位会变成一个值集合，而非有序的阵列。 这让我们可以知道：

* _是否有26岁的追随者？_

但我们无法取得准确的资料如：

* _是否有26岁的追随者**且名字叫Alex Jones？**_

关联内部对象可解决此类问题，我们称之为_嵌套_对象，我们之後会在嵌套对象中提到它。

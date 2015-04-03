### 根对象

映射的最高一层被称为 _根对象_，它可能包含下面几项：

* 一个 _properties_ 节点，列出了文档中可能包含的每个字段的映射

* 多个元数据字段，每一个都以下划线开头，例如 `_type`, `_id` 和 `_source`

* 设置项，控制如何动态处理新的字段，例如 `analyzer`, `dynamic_date_formats` 和 `dynamic_templates`。

* 其他设置，可以同时应用在根对象和其他 `object` 类型的字段上，例如 `enabled`, `dynamic` 和 `include_in_all`

### 属性

我们已经在【核心字段】和【复合核心字段】章节中介绍过文档字段和属性的三个最重要的设置：

`type`：
  字段的数据类型，例如 `string` 和 `date`

`index`：
  字段是否应当被当成全文来搜索（`analyzed`），或被当成一个准确的值（`not_analyzed`），还是完全不可被搜索（`no`）

`analyzer`：
  确定在索引和或搜索时全文字段使用的 `分析器`。

我们将在下面的章节中介绍其他字段，例如 `ip`, `geo_point` 和 `geo_shape`

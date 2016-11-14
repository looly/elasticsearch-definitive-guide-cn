### 动态映射

当 Elasticsearch 处理一个位置的字段时，它通过【动态映射】来确定字段的数据类型且自动将该字段加到类型映射中。

有时这是理想的行为，有时却不是。或许你不知道今后会有哪些字段加到文档中，但是你希望它们能自动被索引。或许你仅仅想忽略它们。特别是当你使用 Elasticsearch 作为主数据源时，你希望未知字段能抛出一个异常来警示你。

幸运的是，你可以通过 `dynamic` 设置来控制这些行为，它接受下面几个选项：

`true`：自动添加字段（默认）

`false`：忽略字段

`strict`：当遇到未知字段时抛出异常

`dynamic` 设置可以用在根对象或任何 `object` 对象上。你可以将 `dynamic` 默认设置为 `strict`，而在特定内部对象上启用它：

```
PUT /my_index
{
    "mappings": {
        "my_type": {
            "dynamic":      "strict", <1>
            "properties": {
                "title":  { "type": "string"},
                "stash":  {
                    "type":     "object",
                    "dynamic":  true <2>
                }
            }
        }
    }
}
```

<!-- SENSE: 070_Index_Mgmt/35_Dynamic_mapping.json -->

<1> 当遇到未知字段时，`my_type` 对象将会抛出异常

<2> `stash` 对象会自动创建字段

通过这个映射，你可以添加一个新的可搜索字段到 `stash` 对象中：

```
PUT /my_index/my_type/1
{
    "title":   "This doc adds a new field",
    "stash": { "new_field": "Success!" }
}
```

<!-- SENSE: 070_Index_Mgmt/35_Dynamic_mapping.json -->

但是在顶层做同样的操作则会失败：

```
PUT /my_index/my_type/1
{
    "title":     "This throws a StrictDynamicMappingException",
    "new_field": "Fail!"
}
```

<!-- SENSE: 070_Index_Mgmt/35_Dynamic_mapping.json -->

备注：将 `dynamic` 设置成 `false` 完全不会修改 `_source` 字段的内容。`_source` 将仍旧保持你索引时的完整 JSON 文档。然而，没有被添加到映射的未知字段将不可被搜索。

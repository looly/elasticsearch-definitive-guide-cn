### 索引设置

你可以通过很多种方式来自定义索引行为，你可以阅读[Index Modules reference documentation](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/_index_settings.html#_index_settings)，但是：

提示: Elasticsearch 提供了优化好的默认配置。除非你明白这些配置的行为和为什么要这么做，请不要修改这些配置。

下面是两个最重要的设置：

`number_of_shards`

    定义一个索引的主分片个数，默认值是 `5`。这个配置在索引创建后不能修改。

`number_of_replicas`

    每个主分片的复制分片个数，默认是 `1`。这个配置可以随时在活跃的索引上修改。

例如，我们可以创建只有一个主分片，没有复制分片的小索引。

```
PUT /my_temp_index
{
    "settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    }
}
```

<!-- SENSE: 070_Index_Mgmt/10_Settings.json -->

然后，我们可以用 `update-index-settings` API 动态修改复制分片个数：

```
PUT /my_temp_index/_settings
{
    "number_of_replicas": 1
}
```

<!-- SENSE: 070_Index_Mgmt/10_Settings.json -->

---
title: "路由元数据字段（_routing）"
date: 0001-01-01
summary: "_routing 元数据字段 #  Easysearch 使用哈希算法将文档路由到索引中的特定分片。默认情况下，文档的 _id 字段用作路由值，但您也可以为每个文档指定自定义路由值。
默认路由 #  以下是 Easysearch 的默认路由公式。_routing 值是文档的 _id。
相关指南（先读这些） #    映射基础  元数据字段  shard_num = hash(_routing) % num_primary_shards 自定义路由 #  您可以在索引文档时指定自定义路由值，如以下示例所示：
PUT sample-index1/_doc/1?routing=JohnDoe1 { &#34;title&#34;: &#34;This is a document&#34; } 在此示例中，文档使用的路由值是 JohnDoe1 而不是默认的 _id 。
在检索、删除或更新文档时，您必须提供相同的路由值，如以下示例所示：
GET sample-index1/_doc/1?routing=JohnDoe1 通过路由查询 #  您可以使用 _routing 字段根据文档的路由值进行查询，如以下示例所示。此查询仅搜索与 JohnDoe1 路由值关联的分片：
GET sample-index1/_search { &#34;query&#34;: { &#34;terms&#34;: { &#34;_routing&#34;: [ &#34;JohnDoe1&#34; ] } } } 设置路由为必需项 #  您可以使索引上的所有 CRUD 操作都必需提供路由值，如以下示例。如果您尝试在不提供路由值的情况下索引文档，Easysearch 将抛出异常。"
---


# _routing 元数据字段

Easysearch 使用哈希算法将文档路由到索引中的特定分片。默认情况下，文档的 `_id` 字段用作路由值，但您也可以为每个文档指定自定义路由值。

## 默认路由

以下是 Easysearch 的默认路由公式。`_routing` 值是文档的 `_id`。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [元数据字段]({{< relref "/docs/features/mapping-and-analysis/metadata-field/_index.md" >}})

```
shard_num = hash(_routing) % num_primary_shards
```

## 自定义路由

您可以在索引文档时指定自定义路由值，如以下示例所示：

```
PUT sample-index1/_doc/1?routing=JohnDoe1
{
  "title": "This is a document"
}
```

在此示例中，文档使用的路由值是 `JohnDoe1` 而不是默认的 `_id` 。

在检索、删除或更新文档时，您必须提供相同的路由值，如以下示例所示：

```
GET sample-index1/_doc/1?routing=JohnDoe1
```

## 通过路由查询

您可以使用 `_routing` 字段根据文档的路由值进行查询，如以下示例所示。此查询仅搜索与 `JohnDoe1` 路由值关联的分片：

```
GET sample-index1/_search
{
  "query": {
    "terms": {
      "_routing": [ "JohnDoe1" ]
    }
  }
}
```

## 设置路由为必需项

您可以使索引上的所有 CRUD 操作都必需提供路由值，如以下示例。如果您尝试在不提供路由值的情况下索引文档，Easysearch 将抛出异常。

```
PUT sample-index2
{
  "mappings": {
    "_routing": {
      "required": true
    }
  }
}
```

## 路由到特定分片组

您可以配置索引将自定义值路由到分片的子集，而不是单个分片。这是通过在创建索引时设置 `index.routing_partition_size` 来实现的。计算分片的公式是 `shard_num = (hash(_routing) + hash(_id)) % routing_partition_size) % num_primary_shards`。

以下示例请求将文档路由到索引中的四个分片之一：

```
PUT sample-index3
{
  "settings": {
    "index.routing_partition_size": 4
  },
  "mappings": {
    "_routing": {
      "required": true
    }
  }
}


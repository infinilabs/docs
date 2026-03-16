---
title: "文档 ID 元数据字段（_id）"
date: 0001-01-01
summary: "_id 元数据字段 #  Easysearch 中的每个文档都有一个唯一的 _id 字段。此字段已被索引，允许你使用 GET API 或 ids 查询 检索文档。
 注意：如果您未提供 _id 值，则 Easysearch 会自动为文档生成一个。
 相关指南（先读这些） #    映射基础  元数据字段  以下示例请求创建一个名为 test-index1 的索引，并添加两个具有不同 _id 值的文档：
PUT test-index1/_doc/1 { &#34;text&#34;: &#34;Document with ID 1&#34; } PUT test-index1/_doc/2?refresh=true { &#34;text&#34;: &#34;Document with ID 2&#34; } 您可以使用 _id 字段查询文档，如以下示例请求所示：
GET test-index1/_search { &#34;query&#34;: { &#34;terms&#34;: { &#34;_id&#34;: [&#34;1&#34;, &#34;2&#34;] } } } 返回 _id 值为 1 和 2 的两个文档："
---


# _id 元数据字段

Easysearch 中的每个文档都有一个唯一的 `_id` 字段。此字段已被索引，允许你使用 GET API 或 [ids 查询]({{< relref "/docs/features/query-dsl/term-based-query/ids.md" >}}) 检索文档。

> **注意**：如果您未提供 `_id` 值，则 Easysearch 会自动为文档生成一个。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [元数据字段]({{< relref "/docs/features/mapping-and-analysis/metadata-field/_index.md" >}})

以下示例请求创建一个名为 `test-index1` 的索引，并添加两个具有不同 `_id` 值的文档：

```
PUT test-index1/_doc/1
{
  "text": "Document with ID 1"
}

PUT test-index1/_doc/2?refresh=true
{
  "text": "Document with ID 2"
}
```

您可以使用 `_id` 字段查询文档，如以下示例请求所示：

```
GET test-index1/_search
{
  "query": {
    "terms": {
      "_id": ["1", "2"]
    }
  }
}
```

返回 `_id` 值为 `1` 和 `2` 的两个文档：

```
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "test-index1",
        "_id": "1",
        "_score": 1,
        "_source": {
          "text": "Document with ID 1"
        }
      },
      {
        "_index": "test-index1",
        "_id": "2",
        "_score": 1,
        "_source": {
          "text": "Document with ID 2"
        }
      }
    ]
  }
```

### `_id` 字段的限制

虽然 `_id` 字段可以在各种查询中使用，但它在聚合、排序和脚本中的使用受到限制。如果您需要对 `_id` 字段进行排序或聚合，建议将 `_id` 内容复制到另一个启用了 `doc_values` 的字段中。


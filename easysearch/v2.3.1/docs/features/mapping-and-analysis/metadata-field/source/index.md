---
title: "源数据元数据字段（_source）"
date: 0001-01-01
summary: "_source 元数据字段 #  _source 字段包含已索引的原始 JSON 文档主体。虽然此字段不可搜索，但它会被存储，以便在执行获取请求（如 get 和 search）时可以返回完整文档。
禁用 _source #  您可以通过将 enabled 参数设置为 false 来禁用 _source 字段，如以下示例所示：
PUT sample-index1 { &#34;mappings&#34;: { &#34;_source&#34;: { &#34;enabled&#34;: false } } }  注意：禁用 _source 字段可能会影响某些功能的可用性，例如 update、update_by_query 和 reindex API，以及使用原始索引文档查询或聚合的能力。
 相关指南（先读这些） #    映射基础  元数据字段  包含或排除某些字段 #  您可以使用 includes 和 excludes 参数选择 _source 字段的内容。如以下示例：
PUT logs { &#34;mappings&#34;: { &#34;_source&#34;: { &#34;includes&#34;: [ &#34;*."
---


# _source 元数据字段

`_source` 字段包含已索引的原始 JSON 文档主体。虽然此字段不可搜索，但它会被存储，以便在执行获取请求（如 `get` 和 `search`）时可以返回完整文档。

## 禁用 _source

您可以通过将 `enabled` 参数设置为 `false` 来禁用 `_source` 字段，如以下示例所示：

```
PUT sample-index1
{
  "mappings": {
    "_source": {
      "enabled": false
    }
  }
}
```

> **注意**：禁用 `_source` 字段可能会影响某些功能的可用性，例如 `update`、`update_by_query` 和 `reindex` API，以及使用原始索引文档查询或聚合的能力。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [元数据字段]({{< relref "/docs/features/mapping-and-analysis/metadata-field/_index.md" >}})

## 包含或排除某些字段

您可以使用 `includes` 和 `excludes` 参数选择 `_source` 字段的内容。如以下示例：

```
PUT logs
{
  "mappings": {
    "_source": {
      "includes": [
        "*.count",
        "meta.*"
      ],
      "excludes": [
        "meta.description",
        "meta.other.*"
      ]
    }
  }
}
```

这些字段不会存储在 `_source` 中，但您仍然可以搜索它们，因为数据仍然被索引。

## 搜索时过滤 _source

除了在映射中配置，您还可以在搜索请求中动态控制 `_source` 返回的字段：

### 禁用 _source 返回

```
GET my_index/_search
{
  "_source": false,
  "query": { "match_all": {} }
}
```

### 只返回指定字段

```
GET my_index/_search
{
  "_source": ["user", "message"],
  "query": { "match_all": {} }
}
```

### 使用 includes/excludes

```
GET my_index/_search
{
  "_source": {
    "includes": ["user.*"],
    "excludes": ["user.password"]
  },
  "query": { "match_all": {} }
}
```

## 禁用 _source 的影响

| 功能                        | _source 启用 | _source 禁用 |
| --------------------------- | ------------ | ------------ |
| `GET /index/_doc/id`        | ✅ 返回文档  | ❌ 无内容     |
| `update` API                | ✅ 正常      | ❌ 不可用     |
| `update_by_query`           | ✅ 正常      | ❌ 不可用     |
| `reindex`                   | ✅ 正常      | ❌ 不可用     |
| `highlight`                 | ✅ 正常      | ⚠️ 需要额外配置 |
| 搜索和聚合                  | ✅ 正常      | ✅ 正常       |

> **建议**：除非有明确的磁盘空间优化需求，否则保持 `_source` 启用。禁用后会导致很多日常运维操作不可用。


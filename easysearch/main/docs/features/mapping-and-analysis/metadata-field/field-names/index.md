---
title: "字段名称元数据字段（_field_names）"
date: 0001-01-01
summary: "_field_names 元数据字段 #  _field_names 字段索引包含非空值的字段名称。可以使用 exists 查询来识别指定字段是否具有非空值的文档。
但是，只有当 doc_values 和 norms 都被禁用时，_field_names 才会索引字段名称。如果启用了 doc_values 或 norms 中的任何一个，则 exists 查询仍然可以工作，但不会依赖 _field_names 字段。
相关指南（先读这些） #    映射基础  元数据字段  映射示例 #  PUT testindex { &#34;mappings&#34;: { &#34;_field_names&#34;: { &#34;enabled&#34;: true }, &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;norms&#34;: false }, &#34;description&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;price&#34;: { &#34;type&#34;: &#34;float&#34;, &#34;doc_values&#34;: false } } } } "
---


# _field_names 元数据字段

`_field_names` 字段索引包含非空值的字段名称。可以使用 `exists` 查询来识别指定字段是否具有非空值的文档。

但是，只有当 `doc_values` 和 `norms` 都被禁用时，`_field_names` 才会索引字段名称。如果启用了 `doc_values` 或 `norms` 中的任何一个，则 `exists` 查询仍然可以工作，但不会依赖 `_field_names` 字段。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [元数据字段]({{< relref "/docs/features/mapping-and-analysis/metadata-field/_index.md" >}})

## 映射示例

```json
PUT testindex
{
  "mappings": {
    "_field_names": {
      "enabled": true
    },
    "properties": {
      "title": {
        "type": "text",
        "norms": false
      },
      "description": {
        "type": "text"
      },
      "price": {
        "type": "float",
        "doc_values": false
      }
    }
  }
}
```


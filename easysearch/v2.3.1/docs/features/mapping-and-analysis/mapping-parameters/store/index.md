---
title: "存储参数（Store）"
date: 0001-01-01
summary: "Store 参数 #  store 参数指定字段值是否应被独立存储，以便可以脱离 _source 单独检索。
默认情况下，字段值在 _source 中存储，但不会被独立存储。当你需要检索某个字段的值时，Easysearch 会加载整个 _source 文档并提取该字段。如果一个文档非常大，而你只需要获取其中一两个小字段的值，使用 store: true 可能更高效。
相关指南（先读这些） #    _source 与字段存储  映射基础  参数选项 #     值 说明     true 字段值被独立存储，可通过 stored_fields 参数单独检索。   false 字段值不独立存储。默认值。    示例 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;store&#34;: true }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;created_at&#34;: { &#34;type&#34;: &#34;date&#34;, &#34;store&#34;: true } } } } 检索独立存储的字段："
---


# Store 参数

`store` 参数指定字段值是否应被独立存储，以便可以脱离 `_source` 单独检索。

默认情况下，字段值在 `_source` 中存储，但不会被独立存储。当你需要检索某个字段的值时，Easysearch 会加载整个 `_source` 文档并提取该字段。如果一个文档非常大，而你只需要获取其中一两个小字段的值，使用 `store: true` 可能更高效。

## 相关指南（先读这些）

- [_source 与字段存储]({{< relref "/docs/fundamentals/source-and-stored-fields.md" >}})
- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})

## 参数选项

| 值 | 说明 |
|----|------|
| `true` | 字段值被独立存储，可通过 `stored_fields` 参数单独检索。 |
| `false` | 字段值不独立存储。**默认值**。 |

## 示例

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "store": true
      },
      "content": {
        "type": "text"
      },
      "created_at": {
        "type": "date",
        "store": true
      }
    }
  }
}
```

检索独立存储的字段：

```json
GET my-index/_search
{
  "stored_fields": ["title", "created_at"],
  "_source": false
}
```

## 适用场景

| 场景 | 建议 |
|------|------|
| 文档很大（如包含大段正文），只需返回少数字段 | 考虑 `store: true` |
| 文档较小，返回大部分字段 | 使用默认 `_source`，无需 `store` |
| 已禁用 `_source`（`"_source": {"enabled": false}`） | 必须用 `store: true` 保存需要返回的字段 |

## 注意事项

- 每个 `store: true` 的字段都会额外占用磁盘空间
- 绝大多数场景下，直接使用 `_source`（配合 `_source` filtering）即可，不需要 `store`
- `store: true` 的字段在 `_source` 中仍然存在（除非禁用了 `_source`）


---
title: "评分规范参数（Norms）"
date: 0001-01-01
summary: "Norms 参数 #  norms 参数控制是否存储字段长度归一化因子，用于相关性评分计算。
在 BM25 评分算法中，字段长度是一个重要因素：短字段中的匹配通常比长字段中的匹配更相关。norms 存储的就是这个字段长度信息。
相关指南（先读这些） #    映射基础  评分基础  参数选项 #     字段类型 默认值 说明     text true 默认启用，用于全文搜索评分   keyword false 默认禁用，keyword 通常用于过滤/聚合    示例 #  禁用 text 字段的 norms：
PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;tags&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;norms&#34;: false } } } } 何时禁用 norms #     场景 建议     字段用于全文搜索，需要相关性排序 保持启用   字段仅用于过滤（filter context） 可禁用，节省空间   字段是日志/标签等，不关心评分 可禁用   多个字段的评分权重由 boost 控制 保持启用    注意事项 #   禁用 norms 可以节省磁盘空间（每个文档每个字段约 1 byte） norms 一旦禁用后无法重新启用，需要重建索引 对于不需要评分的字段（如纯过滤用途），禁用 norms 是安全的优化  "
---


# Norms 参数

`norms` 参数控制是否存储字段长度归一化因子，用于相关性评分计算。

在 BM25 评分算法中，字段长度是一个重要因素：短字段中的匹配通常比长字段中的匹配更相关。norms 存储的就是这个字段长度信息。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [评分基础]({{< relref "/docs/features/fulltext-search/relevance/scoring-basics.md" >}})

## 参数选项

| 字段类型 | 默认值 | 说明 |
|----------|--------|------|
| `text` | `true` | 默认启用，用于全文搜索评分 |
| `keyword` | `false` | 默认禁用，keyword 通常用于过滤/聚合 |

## 示例

禁用 text 字段的 norms：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "tags": {
        "type": "text",
        "norms": false
      }
    }
  }
}
```

## 何时禁用 norms

| 场景 | 建议 |
|------|------|
| 字段用于全文搜索，需要相关性排序 | **保持启用** |
| 字段仅用于过滤（filter context） | 可禁用，节省空间 |
| 字段是日志/标签等，不关心评分 | 可禁用 |
| 多个字段的评分权重由 boost 控制 | 保持启用 |

## 注意事项

- 禁用 norms 可以节省磁盘空间（每个文档每个字段约 1 byte）
- norms **一旦禁用后无法重新启用**，需要重建索引
- 对于不需要评分的字段（如纯过滤用途），禁用 norms 是安全的优化


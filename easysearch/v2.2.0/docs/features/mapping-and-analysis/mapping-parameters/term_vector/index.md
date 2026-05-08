---
title: "词向量参数（Term Vector）"
date: 0001-01-01
summary: "Term Vector 参数 #  term_vector 参数控制是否为字段存储词条向量（Term Vector）信息。词条向量包含词条及其位置、偏移量等信息，供高亮和 More Like This 查询使用。
相关指南 #    全文搜索  analyzer 参数  可选值 #     值 说明     no 默认值。不存储词条向量。   yes 仅存储词条，不含位置和偏移量。   with_positions 存储词条和位置信息。   with_offsets 存储词条和字符偏移量。   with_positions_offsets 存储词条、位置和偏移量。推荐用于快速高亮。   with_positions_payloads 存储词条、位置和有效载荷。   with_positions_offsets_payloads 存储所有信息。    示例 #  为高亮优化 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;term_vector&#34;: &#34;with_positions_offsets&#34; } } } } 使用 with_positions_offsets 可以让 fast vector highlighter（FVH）高亮器直接从词条向量中提取数据，而无需重新分析文档，显著提高高亮性能。"
---


# Term Vector 参数

`term_vector` 参数控制是否为字段存储词条向量（Term Vector）信息。词条向量包含词条及其位置、偏移量等信息，供高亮和 More Like This 查询使用。

## 相关指南

- [全文搜索]({{< relref "/docs/features/fulltext-search/" >}})
- [analyzer 参数]({{< relref "/docs/features/mapping-and-analysis/mapping-parameters/analyzer.md" >}})

## 可选值

| 值 | 说明 |
|----|------|
| `no` | **默认值**。不存储词条向量。 |
| `yes` | 仅存储词条，不含位置和偏移量。 |
| `with_positions` | 存储词条和位置信息。 |
| `with_offsets` | 存储词条和字符偏移量。 |
| `with_positions_offsets` | 存储词条、位置和偏移量。**推荐用于快速高亮**。 |
| `with_positions_payloads` | 存储词条、位置和有效载荷。 |
| `with_positions_offsets_payloads` | 存储所有信息。 |

## 示例

### 为高亮优化

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}
```

使用 `with_positions_offsets` 可以让 `fast vector highlighter`（FVH）高亮器直接从词条向量中提取数据，而无需重新分析文档，显著提高高亮性能。

## 何时启用

| 场景 | 建议 |
|------|------|
| 需要快速高亮大文本字段 | 使用 `with_positions_offsets` |
| 使用 More Like This 查询 | 使用 `yes` 或更高级别 |
| 不需要高亮 | 保持默认 `no` |

## 注意事项

- 存储词条向量会**显著增加索引大小**，通常增加 50%–100%
- 仅对 `text` 类型字段有意义
- 如果只需要普通高亮（`unified` 或 `plain` 高亮器），不必启用词条向量
- 对于 More Like This 查询，如果不存储词条向量，系统会实时分析文本，速度较慢但节省空间


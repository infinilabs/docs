---
title: "加权参数（Boost）"
date: 0001-01-01
summary: "Boost 参数 #   ⚠️ 已弃用：映射级别的 boost 参数已被弃用，将在未来版本中移除。请改为在查询时通过 boost 参数控制字段权重。
 boost 映射参数用于在搜索查询期间增加或减少字段的相关性分数。它允许你在计算文档的整体相关性分数时，对特定字段应用更多或更少的权重。默认值为 1.0。
boost 参数作为字段分数的乘数应用。例如，如果一个字段的 boost 值为 2，则该字段的分数的权重将翻倍。相反，boost 值为 0.5 将使该字段的分数的权重减半。
相关指南（先读这些） #    映射基础  相关性：加权与调参  代码样例 #  以下是在 Easysearch 映射中使用 boost 参数的示例：
PUT my-index1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;boost&#34;: 2 }, &#34;description&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;boost&#34;: 1 }, &#34;tags&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;boost&#34;: 1.5 } } } } 在此示例中，title 字段的提升值为 2，这意味着它对整体相关性分数的权重是描述字段（提升值为 1）的两倍。tags 字段的提升值为 1."
---


# Boost 参数

> ⚠️ **已弃用**：映射级别的 `boost` 参数已被弃用，将在未来版本中移除。请改为在查询时通过 `boost` 参数控制字段权重。

`boost` 映射参数用于在搜索查询期间增加或减少字段的相关性分数。它允许你在计算文档的整体相关性分数时，对特定字段应用更多或更少的权重。默认值为 `1.0`。

`boost` 参数作为字段分数的乘数应用。例如，如果一个字段的 `boost` 值为 `2`，则该字段的分数的权重将翻倍。相反，`boost` 值为 `0.5` 将使该字段的分数的权重减半。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [相关性：加权与调参]({{< relref "/docs/features/fulltext-search/relevance/boosting.md" >}})

## 代码样例

以下是在 Easysearch 映射中使用 `boost` 参数的示例：

```
PUT my-index1
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "boost": 2
      },
      "description": {
        "type": "text",
        "boost": 1
      },
      "tags": {
        "type": "keyword",
        "boost": 1.5
      }
    }
  }
}
```

在此示例中，`title` 字段的提升值为 `2`，这意味着它对整体相关性分数的权重是描述字段（提升值为 `1`）的两倍。`tags` 字段的提升值为 `1.5`，因此它的权重是描述字段的一倍半。

当您想要对某些字段赋予更多权重时，`boost` 参数特别有用。例如，您可能想要将 `title` 字段的权重提升得比 `description` 字段更高，因为标题更能反映文档的相关性。

**推荐做法**：不要在映射中设置 `boost`，而是在查询时使用 `boost` 参数：

```json
GET my-index1/_search
{
  "query": {
    "multi_match": {
      "query": "Easysearch",
      "fields": ["title^2", "description", "tags^1.5"]
    }
  }
}
```

这种方式更灵活，可以随时调整权重而无需重建索引。

`boost` 参数是一个乘法因子，而不是加法因子。这意味着与具有较低权重值的字段相比，具有较高权重值的字段对整体相关性分数的影响将不成比例地大。在使用 `boost` 参数时，建议您从小值（1.5 或 2）开始，并测试其对搜索结果的影响。过高的权重值可能会扭曲相关性分数，并导致意外或不理想的搜索结果。


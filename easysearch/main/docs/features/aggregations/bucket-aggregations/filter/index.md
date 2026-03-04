---
title: "过滤器聚合（Filter）"
date: 0001-01-01
summary: "过滤器聚合 #  filter 聚合是一个查询子句，就像一个搜索查询一样 — match 或 term 或 range。您可以使用 filter 聚合在创建分组之前将整个文档集缩小到特定的文档集。
相关指南（先读这些） #    聚合基础  聚合场景实践  以下示例展示了 avg 聚合在过滤上下文中运行的情况。 avg 聚合仅聚合与 range 查询匹配的文档：
GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;low_value&#34;: { &#34;filter&#34;: { &#34;range&#34;: { &#34;taxful_total_price&#34;: { &#34;lte&#34;: 50 } } }, &#34;aggs&#34;: { &#34;avg_amount&#34;: { &#34;avg&#34;: { &#34;field&#34;: &#34;taxful_total_price&#34; } } } } } } 返回内容
... &#34;aggregations&#34; : { &#34;low_value&#34; : { &#34;doc_count&#34; : 1633, &#34;avg_amount&#34; : { &#34;value&#34; : 38."
---


# 过滤器聚合

`filter` 聚合是一个查询子句，就像一个搜索查询一样 — `match` 或 `term` 或 `range`。您可以使用 `filter` 聚合在创建分组之前将整个文档集缩小到特定的文档集。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

以下示例展示了 `avg` 聚合在过滤上下文中运行的情况。 `avg` 聚合仅聚合与 `range` 查询匹配的文档：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "low_value": {
      "filter": {
        "range": {
          "taxful_total_price": {
            "lte": 50
          }
        }
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

返回内容

```
...
"aggregations" : {
  "low_value" : {
    "doc_count" : 1633,
    "avg_amount" : {
      "value" : 38.363175998928355
    }
  }
 }
}
```

## filter vs. 查询 + 聚合

`filter` 聚合与在查询中使用 `bool.filter` 有所不同：

| 方式              | 作用范围                                   | 典型用途                              |
| ----------------- | ------------------------------------------ | ------------------------------------- |
| 查询中的 `filter` | 过滤整个搜索的文档集（影响所有聚合）       | 全局过滤条件                          |
| `filter` 聚合     | 仅在该聚合内过滤（不影响其他聚合）         | 对比分析（例如同时看"全部"和"高价"）  |

### 对比示例

以下请求同时计算全部商品均价和高价商品（>100）均价：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "all_avg_price": {
      "avg": {
        "field": "taxful_total_price"
      }
    },
    "expensive_avg_price": {
      "filter": {
        "range": {
          "taxful_total_price": {
            "gt": 100
          }
        }
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

> **提示**：如需同时使用多个过滤条件分桶，请使用 [filters 聚合]({{< relref "./filters.md" >}})。


---
title: "全局聚合（Global）"
date: 0001-01-01
summary: "全局聚合 #  global 聚合让你能跳出过滤聚合的聚合上下文。即使你包含了一个缩小文档集的过滤查询，global 聚合仍然对所有文档进行聚合，就好像过滤查询不存在一样。它忽略 filter 聚合，并隐式地假设 match_all 查询。
相关指南（先读这些） #    聚合基础  聚合场景实践  以下示例返回索引中所有文档的 taxful_total_price 字段的 avg 值：
GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;query&#34;: { &#34;range&#34;: { &#34;taxful_total_price&#34;: { &#34;lte&#34;: 50 } } }, &#34;aggs&#34;: { &#34;total_avg_amount&#34;: { &#34;global&#34;: {}, &#34;aggs&#34;: { &#34;avg_price&#34;: { &#34;avg&#34;: { &#34;field&#34;: &#34;taxful_total_price&#34; } } } } } } 返回内容
... &#34;aggregations&#34; : { &#34;total_avg_amount&#34; : { &#34;doc_count&#34; : 4675, &#34;avg_price&#34; : { &#34;value&#34; : 75."
---


# 全局聚合

`global` 聚合让你能跳出过滤聚合的聚合上下文。即使你包含了一个缩小文档集的过滤查询，`global` 聚合仍然对所有文档进行聚合，就好像过滤查询不存在一样。它忽略 `filter` 聚合，并隐式地假设 `match_all` 查询。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

以下示例返回索引中所有文档的 `taxful_total_price` 字段的 `avg` 值：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "range": {
      "taxful_total_price": {
        "lte": 50
      }
    }
  },
  "aggs": {
    "total_avg_amount": {
      "global": {},
      "aggs": {
        "avg_price": {
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
  "total_avg_amount" : {
    "doc_count" : 4675,
    "avg_price" : {
      "value" : 75.05542864304813
    }
  }
 }
}
```

你可以看到， `taxful_total_price` 字段的平均值是 75.05，而不是 `filter` 示例中查询匹配时看到的 38.36。

## 典型使用场景

### 对比分析：子集 vs. 全局

`global` 聚合最常见的用途是将某个查询子集的统计值与全局统计值进行对比。例如，同时查看低价商品均价和全部商品均价：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "range": {
      "taxful_total_price": { "lte": 50 }
    }
  },
  "aggs": {
    "filtered_avg": {
      "avg": { "field": "taxful_total_price" }
    },
    "all_products": {
      "global": {},
      "aggs": {
        "global_avg": {
          "avg": { "field": "taxful_total_price" }
        }
      }
    }
  }
}
```

返回结果中，`filtered_avg` 反映了查询命中文档（价格 ≤50）的均价，而 `all_products.global_avg` 反映了所有文档的均价。

> **注意**：`global` 聚合只能作为顶层聚合使用，不能嵌套在其他聚合内部。


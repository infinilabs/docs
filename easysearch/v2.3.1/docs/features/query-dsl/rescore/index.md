---
title: "查询重打分"
date: 0001-01-01
description: "Rescore API：对 Top-N 结果用更精细的查询重新打分，兼顾性能与精度。"
summary: "查询重打分 #  查询重打分（Rescore）允许你在初始查询返回 Top-N 结果后，用更复杂、更精细的查询对这些结果重新打分。这是一种二阶段打分策略：第一阶段用快速查询（如 match）粗筛，第二阶段用代价更高的查询（如 match_phrase、function_score）精排。
使用场景 #   短语精排：先用 match 召回，再用 match_phrase 提升短语完全匹配的文档 复杂评分：初始召回后，对 Top 结果做 function_score 或 script_score 精细计算 向量混排：用 BM25 召回后，对 Top-N 用 kNN 重打分  基础用法 #  GET shakespeare/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;text_entry&#34;: &#34;to be or not to be&#34; } }, &#34;rescore&#34;: { &#34;window_size&#34;: 100, &#34;query&#34;: { &#34;rescore_query&#34;: { &#34;match_phrase&#34;: { &#34;text_entry&#34;: { &#34;query&#34;: &#34;to be or not to be&#34;, &#34;slop&#34;: 2 } } }, &#34;query_weight&#34;: 0."
---


# 查询重打分

查询重打分（Rescore）允许你在初始查询返回 Top-N 结果后，用更复杂、更精细的查询对这些结果重新打分。这是一种**二阶段打分策略**：第一阶段用快速查询（如 `match`）粗筛，第二阶段用代价更高的查询（如 `match_phrase`、`function_score`）精排。

## 使用场景

- 短语精排：先用 `match` 召回，再用 `match_phrase` 提升短语完全匹配的文档
- 复杂评分：初始召回后，对 Top 结果做 `function_score` 或 `script_score` 精细计算
- 向量混排：用 BM25 召回后，对 Top-N 用 kNN 重打分

## 基础用法

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "to be or not to be"
    }
  },
  "rescore": {
    "window_size": 100,
    "query": {
      "rescore_query": {
        "match_phrase": {
          "text_entry": {
            "query": "to be or not to be",
            "slop": 2
          }
        }
      },
      "query_weight": 0.7,
      "rescore_query_weight": 1.2
    }
  }
}
```

## 参数说明

### 顶层参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `window_size` | Integer | 10 | 每个分片上参与重打分的 Top-N 文档数 |
| `query` | Object | 必填 | 重打分查询配置 |

### query 对象参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `rescore_query` | Query DSL | 必填 | 用于重打分的查询（任意 Query DSL） |
| `query_weight` | Float | 1.0 | 原始查询分数的权重 |
| `rescore_query_weight` | Float | 1.0 | 重打分查询分数的权重 |
| `score_mode` | String | `total` | 原始分数与重打分分数的合并方式 |

### score_mode 可选值

| 值 | 公式 | 说明 |
|------|------|------|
| `total`（或 `sum`） | $\text{original} \times w_1 + \text{rescore} \times w_2$ | 加权求和（默认） |
| `avg` | $\frac{\text{original} \times w_1 + \text{rescore} \times w_2}{2}$ | 加权平均 |
| `max` | $\max(\text{original} \times w_1,\ \text{rescore} \times w_2)$ | 取最大值 |
| `min` | $\min(\text{original} \times w_1,\ \text{rescore} \times w_2)$ | 取最小值 |
| `multiply`（或 `product`） | $\text{original} \times w_1 \times \text{rescore} \times w_2$ | 加权乘积 |

## 多级重打分

可以指定多个 rescore 对象组成数组，按顺序依次执行：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "love"
    }
  },
  "rescore": [
    {
      "window_size": 200,
      "query": {
        "rescore_query": {
          "match_phrase": {
            "text_entry": {
              "query": "love",
              "slop": 1
            }
          }
        },
        "query_weight": 0.7,
        "rescore_query_weight": 1.5
      }
    },
    {
      "window_size": 50,
      "query": {
        "rescore_query": {
          "function_score": {
            "script_score": {
              "script": "_score * doc['popularity'].value"
            }
          }
        },
        "score_mode": "multiply"
      }
    }
  ]
}
```

> 每一级重打分都在上一级的结果基础上执行。

## 性能建议

- `window_size` 越大，精度越高但开销越大。建议从 100~500 开始调试
- 重打分查询不影响召回率（只影响已召回文档的排序）
- 重打分不支持与 `sort` 一起使用（需要按 `_score` 排序才有意义）
- 每个分片独立执行重打分，因此实际重打分文档数 = `window_size × 分片数`


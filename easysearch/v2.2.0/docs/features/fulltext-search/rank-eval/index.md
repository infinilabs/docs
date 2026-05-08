---
title: "排名评估（Rank Eval）"
date: 0001-01-01
summary: "排名评估（Rank Eval） #  排名评估 API（_rank_eval）允许您使用一组预定义的查询和已知相关文档来评估搜索结果的排名质量。这对于调优搜索相关性、对比不同查询策略的效果非常有用。
基本概念 #  排名评估的核心流程：
 定义请求集：一组代表性的搜索查询 标注相关文档：为每个查询标注哪些文档是相关的（及其相关程度） 选择评估指标：如 Precision、Recall、DCG 等 执行评估：Easysearch 运行查询并根据标注计算指标得分  基本用法 #  GET my_index/_rank_eval { &#34;requests&#34;: [ { &#34;id&#34;: &#34;query_1&#34;, &#34;request&#34;: { &#34;query&#34;: { &#34;match&#34;: { &#34;title&#34;: &#34;搜索引擎&#34; } } }, &#34;ratings&#34;: [ { &#34;_index&#34;: &#34;my_index&#34;, &#34;_id&#34;: &#34;doc1&#34;, &#34;rating&#34;: 3 }, { &#34;_index&#34;: &#34;my_index&#34;, &#34;_id&#34;: &#34;doc2&#34;, &#34;rating&#34;: 2 }, { &#34;_index&#34;: &#34;my_index&#34;, &#34;_id&#34;: &#34;doc3&#34;, &#34;rating&#34;: 0 } ] }, { &#34;id&#34;: &#34;query_2&#34;, &#34;request&#34;: { &#34;query&#34;: { &#34;match&#34;: { &#34;title&#34;: &#34;全文检索&#34; } } }, &#34;ratings&#34;: [ { &#34;_index&#34;: &#34;my_index&#34;, &#34;_id&#34;: &#34;doc4&#34;, &#34;rating&#34;: 3 }, { &#34;_index&#34;: &#34;my_index&#34;, &#34;_id&#34;: &#34;doc5&#34;, &#34;rating&#34;: 1 } ] } ], &#34;metric&#34;: { &#34;precision&#34;: { &#34;k&#34;: 10, &#34;relevant_rating_threshold&#34;: 1 } } } 请求参数说明 #     字段 说明     requests 评估请求数组   requests[]."
---


# 排名评估（Rank Eval）

排名评估 API（`_rank_eval`）允许您使用一组预定义的查询和已知相关文档来评估搜索结果的排名质量。这对于调优搜索相关性、对比不同查询策略的效果非常有用。

## 基本概念

排名评估的核心流程：

1. **定义请求集**：一组代表性的搜索查询
2. **标注相关文档**：为每个查询标注哪些文档是相关的（及其相关程度）
3. **选择评估指标**：如 Precision、Recall、DCG 等
4. **执行评估**：Easysearch 运行查询并根据标注计算指标得分

## 基本用法

```json
GET my_index/_rank_eval
{
  "requests": [
    {
      "id": "query_1",
      "request": {
        "query": { "match": { "title": "搜索引擎" } }
      },
      "ratings": [
        { "_index": "my_index", "_id": "doc1", "rating": 3 },
        { "_index": "my_index", "_id": "doc2", "rating": 2 },
        { "_index": "my_index", "_id": "doc3", "rating": 0 }
      ]
    },
    {
      "id": "query_2",
      "request": {
        "query": { "match": { "title": "全文检索" } }
      },
      "ratings": [
        { "_index": "my_index", "_id": "doc4", "rating": 3 },
        { "_index": "my_index", "_id": "doc5", "rating": 1 }
      ]
    }
  ],
  "metric": {
    "precision": {
      "k": 10,
      "relevant_rating_threshold": 1
    }
  }
}
```

### 请求参数说明

| 字段 | 说明 |
|------|------|
| `requests` | 评估请求数组 |
| `requests[].id` | 查询的唯一标识，用于在结果中关联 |
| `requests[].request` | 搜索请求体（与普通 `_search` 相同） |
| `requests[].ratings` | 该查询对应的文档相关性标注 |
| `ratings[].rating` | 相关性分数，通常 0=不相关，1=部分相关，2=相关，3=非常相关 |
| `metric` | 使用的评估指标 |

## 评估指标

### Precision at K（精确率）

计算前 K 个结果中相关文档的比例。

```json
"metric": {
  "precision": {
    "k": 10,
    "relevant_rating_threshold": 1,
    "ignore_unlabeled": false
  }
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `k` | 10 | 评估前 K 个结果 |
| `relevant_rating_threshold` | 1 | rating 大于等于此值视为相关 |
| `ignore_unlabeled` | false | 是否忽略未标注的文档（true=只考虑已标注的文档） |

**公式**：$P@K = \frac{\text{前 K 个结果中的相关文档数}}{K}$

### Recall at K（召回率）

计算前 K 个结果中找到了多少已知的相关文档。

```json
"metric": {
  "recall": {
    "k": 20,
    "relevant_rating_threshold": 1
  }
}
```

**公式**：$R@K = \frac{\text{前 K 个结果中的相关文档数}}{\text{全部已知相关文档数}}$

### Mean Reciprocal Rank（MRR，平均倒数排名）

计算第一个相关文档出现位置的倒数的平均值。适合评估"用户只关心第一个相关结果"的场景。

```json
"metric": {
  "mean_reciprocal_rank": {
    "k": 10,
    "relevant_rating_threshold": 1
  }
}
```

**公式**：$MRR = \frac{1}{|Q|} \sum_{i=1}^{|Q|} \frac{1}{rank_i}$

其中 $rank_i$ 是第 $i$ 个查询中第一个相关文档的排名位置。

### DCG / nDCG（折损累计增益）

DCG 考虑了文档的相关程度和位置，越相关的文档排名越靠前得分越高。nDCG 是归一化后的 DCG（0-1 之间）。

```json
"metric": {
  "dcg": {
    "k": 10,
    "normalize": true
  }
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `k` | 10 | 评估前 K 个结果 |
| `normalize` | false | 设为 true 使用 nDCG（归一化 DCG） |

**公式**：$DCG@K = \sum_{i=1}^{K} \frac{2^{rel_i} - 1}{\log_2(i+1)}$

$nDCG@K = \frac{DCG@K}{IDCG@K}$

### Expected Reciprocal Rank（ERR，期望倒数排名）

考虑了用户浏览行为的级联模型——用户在找到满意结果后会停止浏览。

```json
"metric": {
  "expected_reciprocal_rank": {
    "maximum_relevance": 3,
    "k": 10
  }
}
```

| 参数 | 说明 |
|------|------|
| `maximum_relevance` | 最大相关性分数（用于将 rating 映射到满意度概率） |
| `k` | 评估前 K 个结果 |

## 响应解读

```json
{
  "metric_score": 0.75,
  "details": {
    "query_1": {
      "metric_score": 0.8,
      "unrated_docs": [
        { "_index": "my_index", "_id": "doc10" }
      ],
      "hits": [
        { "hit": { "_index": "my_index", "_id": "doc1" }, "rating": 3 },
        { "hit": { "_index": "my_index", "_id": "doc2" }, "rating": 2 },
        { "hit": { "_index": "my_index", "_id": "doc10" }, "rating": null }
      ]
    },
    "query_2": {
      "metric_score": 0.7,
      "unrated_docs": [],
      "hits": [...]
    }
  },
  "failures": {}
}
```

| 字段 | 说明 |
|------|------|
| `metric_score` | 所有查询的综合得分（平均值） |
| `details.<query_id>.metric_score` | 单个查询的得分 |
| `details.<query_id>.unrated_docs` | 出现在结果中但未标注的文档 |
| `details.<query_id>.hits` | 结果文档列表及其 rating |
| `failures` | 执行失败的查询 |

## 使用模板简化请求

当多个查询结构相似时，可以使用搜索模板减少重复：

```json
GET my_index/_rank_eval
{
  "templates": [
    {
      "id": "match_template",
      "template": {
        "source": {
          "query": {
            "match": {
              "{{field}}": "{{query}}"
            }
          }
        }
      }
    }
  ],
  "requests": [
    {
      "id": "query_1",
      "template_id": "match_template",
      "params": {
        "field": "title",
        "query": "搜索引擎"
      },
      "ratings": [
        { "_index": "my_index", "_id": "doc1", "rating": 3 }
      ]
    },
    {
      "id": "query_2",
      "template_id": "match_template",
      "params": {
        "field": "title",
        "query": "全文检索"
      },
      "ratings": [
        { "_index": "my_index", "_id": "doc4", "rating": 3 }
      ]
    }
  ],
  "metric": {
    "dcg": { "k": 10, "normalize": true }
  }
}
```

## 最佳实践

1. **选择代表性查询**：选择覆盖不同查询类型（高频、长尾、精确、模糊）的查询样本
2. **多人标注取平均**：避免单一标注者的偏差，理想情况下让多人独立标注后取平均
3. **使用 nDCG**：nDCG 同时考虑了排名位置和相关程度，是最全面的评估指标
4. **定期重新评估**：索引数据变化后，定期重新运行排名评估
5. **A/B 对比**：修改查询策略前后分别运行评估，量化改进效果
6. **关注 unrated_docs**：如果很多结果文档未被标注，说明标注集覆盖不足，需要补充


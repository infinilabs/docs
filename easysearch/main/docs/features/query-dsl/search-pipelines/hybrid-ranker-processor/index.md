---
title: "混合搜索排序处理器"
date: 0001-01-01
summary: "混合搜索排序处理器（Hybrid Ranker Processor） #   需要 AI 插件
 hybrid_ranker_processor 是一种搜索阶段结果处理器（search phase results processor），运行于 Query 阶段和 Fetch 阶段之间，用于对混合搜索（hybrid 查询）返回的多路子查询得分进行归一化和融合，生成统一的最终得分排序。
工作原理 #  在混合搜索中，多个子查询（如 BM25 关键词查询 + KNN 向量查询）各自返回独立的得分。这些得分的量纲和分布可能完全不同，无法直接比较。hybrid_ranker_processor 通过以下步骤解决这个问题：
 得分归一化：将各子查询的得分归一化到可比较的范围 得分融合：使用融合算法（如 RRF）将多路归一化后的得分合并为统一的最终得分 根据最终得分重新排序搜索结果  RRF 算法 #  默认使用 Reciprocal Rank Fusion (RRF) 算法，其核心思想是基于排名而非原始分数进行融合：
$$\text{RRF}(d) = \sum_{i=1}^{n} \frac{1}{k + r_i(d)}$$
其中 $k$ 为常数（默认 60），$r_i(d)$ 为文档 $d$ 在第 $i$ 路已返回候选中的跨分片全局排名。
hybrid 查询必须选择包含一个有效 hybrid_ranker_processor 的搜索管道。未选择管道、选择 _none、空管道或重复配置排序处理器会被拒绝。普通非 hybrid 查询使用同一管道时不会自动执行 RRF。
当前每路候选窗口与 from + size 相关，不支持独立 rank_window_size 或加权 RRF；改变分页窗口可能改变候选并集。hybrid 不支持字段排序、search_after、collapse、scroll 和 rescore，这些组合会明确报错。"
---


# 混合搜索排序处理器（Hybrid Ranker Processor）

> **需要 AI 插件**

`hybrid_ranker_processor` 是一种搜索阶段结果处理器（search phase results processor），运行于 Query 阶段和 Fetch 阶段之间，用于对混合搜索（`hybrid` 查询）返回的多路子查询得分进行归一化和融合，生成统一的最终得分排序。

## 工作原理

在混合搜索中，多个子查询（如 BM25 关键词查询 + KNN 向量查询）各自返回独立的得分。这些得分的量纲和分布可能完全不同，无法直接比较。`hybrid_ranker_processor` 通过以下步骤解决这个问题：

1. **得分归一化**：将各子查询的得分归一化到可比较的范围
2. **得分融合**：使用融合算法（如 RRF）将多路归一化后的得分合并为统一的最终得分
3. 根据最终得分重新排序搜索结果

### RRF 算法

默认使用 **Reciprocal Rank Fusion (RRF)** 算法，其核心思想是基于排名而非原始分数进行融合：

$$\text{RRF}(d) = \sum_{i=1}^{n} \frac{1}{k + r_i(d)}$$

其中 $k$ 为常数（默认 60），$r_i(d)$ 为文档 $d$ 在第 $i$ 路已返回候选中的跨分片全局排名。

`hybrid` 查询必须选择包含一个有效 `hybrid_ranker_processor` 的搜索管道。未选择管道、选择 `_none`、空管道或重复配置排序处理器会被拒绝。普通非 `hybrid` 查询使用同一管道时不会自动执行 RRF。

当前每路候选窗口与 `from + size` 相关，不支持独立 `rank_window_size` 或加权 RRF；改变分页窗口可能改变候选并集。`hybrid` 不支持字段排序、`search_after`、`collapse`、scroll 和 rescore，这些组合会明确报错。

## 请求体字段

| 字段                     | 类型    | 是否必填 | 说明                                   |
| ------------------------ | ------- | -------- | -------------------------------------- |
| `combination`            | Object  | 否       | 得分融合配置                           |
| `combination.technique`  | String  | 否       | 融合算法名称。默认 `rrf`               |
| `combination.parameters` | Object  | 否       | 融合算法的参数                         |
| `combination.parameters.rank_constant` | Integer | 否 | 默认60，范围1–10000。放在其他层级或拼错参数会报错 |
| `tag`                    | String  | 否       | 处理器标识标签                         |
| `description`            | String  | 否       | 处理器描述                             |
| `ignore_failure`         | Boolean | 否       | 此排序处理器失败会终止搜索，不能依赖忽略失败来返回未融合结果 |

## 示例

### 基本用法

```json
PUT /_search/pipeline/my_hybrid_pipeline
{
  "rerank_processors": [
    {
      "hybrid_ranker_processor": {
        "tag": "ranker",
        "description": "RRF 混合排序"
      }
    }
  ]
}
```

### 完整混合搜索管道

通常与 `semantic_query_enricher` 和 `hybrid_score_explanation` 配合使用：

```json
PUT /_search/pipeline/full_hybrid_pipeline
{
  "rewrite_processors": [
    {
      "semantic_query_enricher": {
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings",
        "vendor": "openai",
        "api_key": "sk-xxxxxxxx",
        "default_model_id": "text-embedding-v3"
      }
    }
  ],
  "rerank_processors": [
    {
      "hybrid_ranker_processor": {
        "tag": "ranker",
        "description": "使用 RRF 进行混合排序",
        "combination": {
          "technique": "rrf",
          "parameters": { "rank_constant": 60 }
        }
      }
    }
  ],
  "enrich_processors": [
    {
      "hybrid_score_explanation": {
        "tag": "explanation",
        "description": "添加混合评分解释"
      }
    }
  ]
}
```

执行混合搜索：

```json
GET /my_index/_search?search_pipeline=full_hybrid_pipeline
{
  "explain": true,
  "query": {
    "hybrid": {
      "queries": [
        {
          "match": {
            "content": "集群安全配置"
          }
        },
        {
          "semantic": {
            "embedding": {
              "query_text": "如何配置集群安全"
            }
          }
        }
      ]
    }
  }
}
```

管道将自动完成：语义查询增强 → BM25 + KNN 并行检索 → RRF 得分融合 → 返回统一排序结果。

## 相关文档

- [混合搜索]({{< relref "/docs/integrations/ai/hybrid-search.md" >}})：完整的混合搜索工作流和示例
- [语义查询增强处理器]({{< relref "semantic-query-enricher-processor.md" >}})：自动注入 Embedding 模型信息
- [混合评分解释处理器]({{< relref "hybrid-score-explanation-processor.md" >}})：为混合搜索结果添加评分解释


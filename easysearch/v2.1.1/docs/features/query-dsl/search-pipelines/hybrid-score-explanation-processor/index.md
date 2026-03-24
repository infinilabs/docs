---
title: "混合评分解释处理器"
date: 0001-01-01
summary: "混合评分解释处理器（Hybrid Score Explanation Processor） #   需要 AI 插件
 hybrid_score_explanation 响应处理器用于为混合搜索结果添加评分解释信息。当与 hybrid_ranker_processor 配合使用时，该处理器将归一化和得分融合的详细过程附加到每个搜索命中结果中，帮助用户理解最终得分的来源。
工作原理 #  在混合搜索管道中，hybrid_ranker_processor 在得分融合过程中会将详细的计算信息（各子查询的原始得分、归一化方式、融合方式）记录到管道处理上下文中。hybrid_score_explanation 处理器在响应阶段读取这些信息，并将其合并到每个搜索命中结果的 _explanation 字段中。
解释信息包括：
 归一化详情：使用的归一化技术及每个子查询得分的归一化结果 得分融合详情：使用的融合算法（如 RRF）及各路得分的融合过程  请求体字段 #     字段 类型 是否必填 说明     tag String 否 处理器标识标签   description String 否 处理器描述   ignore_failure Boolean 否 处理器失败时是否继续执行。默认 false    示例 #  基本用法 #  PUT /_search/pipeline/my_explain_pipeline { &#34;phase_results_processors&#34;: [ { &#34;hybrid_ranker_processor&#34;: { &#34;tag&#34;: &#34;ranker&#34;, &#34;description&#34;: &#34;RRF 混合排序&#34; } } ], &#34;response_processors&#34;: [ { &#34;hybrid_score_explanation&#34;: { &#34;tag&#34;: &#34;explanation&#34;, &#34;description&#34;: &#34;添加混合评分解释&#34; } } ] } 返回结果示例 #  使用上述管道执行混合搜索后，每个命中结果的 _explanation 中会包含类似以下的评分解释："
---


# 混合评分解释处理器（Hybrid Score Explanation Processor）

> **需要 AI 插件**

`hybrid_score_explanation` 响应处理器用于为混合搜索结果添加评分解释信息。当与 `hybrid_ranker_processor` 配合使用时，该处理器将归一化和得分融合的详细过程附加到每个搜索命中结果中，帮助用户理解最终得分的来源。

## 工作原理

在混合搜索管道中，`hybrid_ranker_processor` 在得分融合过程中会将详细的计算信息（各子查询的原始得分、归一化方式、融合方式）记录到管道处理上下文中。`hybrid_score_explanation` 处理器在响应阶段读取这些信息，并将其合并到每个搜索命中结果的 `_explanation` 字段中。

解释信息包括：
- **归一化详情**：使用的归一化技术及每个子查询得分的归一化结果
- **得分融合详情**：使用的融合算法（如 RRF）及各路得分的融合过程

## 请求体字段

| 字段 | 类型 | 是否必填 | 说明 |
|------|------|---------|------|
| `tag` | String | 否 | 处理器标识标签 |
| `description` | String | 否 | 处理器描述 |
| `ignore_failure` | Boolean | 否 | 处理器失败时是否继续执行。默认 `false` |

## 示例

### 基本用法

```json
PUT /_search/pipeline/my_explain_pipeline
{
  "phase_results_processors": [
    {
      "hybrid_ranker_processor": {
        "tag": "ranker",
        "description": "RRF 混合排序"
      }
    }
  ],
  "response_processors": [
    {
      "hybrid_score_explanation": {
        "tag": "explanation",
        "description": "添加混合评分解释"
      }
    }
  ]
}
```

### 返回结果示例

使用上述管道执行混合搜索后，每个命中结果的 `_explanation` 中会包含类似以下的评分解释：

```json
{
  "hits": {
    "hits": [
      {
        "_index": "my_index",
        "_id": "1",
        "_score": 0.016393442,
        "_explanation": {
          "value": 0.016393442,
          "description": "combined score of:",
          "details": [
            {
              "value": 0.016393442,
              "description": "rrf combination of:",
              "details": [
                {
                  "value": 0.016129032,
                  "description": "rrf normalization of:",
                  "details": [
                    {
                      "value": 1.0,
                      "description": "keyword query score"
                    }
                  ]
                },
                {
                  "value": 0.016393442,
                  "description": "rrf normalization of:",
                  "details": [
                    {
                      "value": 0.95,
                      "description": "vector query score"
                    }
                  ]
                }
              ]
            }
          ]
        }
      }
    ]
  }
}
```

## 注意事项

- 该处理器通常与 `hybrid_ranker_processor` 配合使用，单独使用时无评分解释信息可供附加
- 使用 `explain: true` 参数可以同时获取搜索引擎自身的评分解释和混合搜索的评分解释

## 相关文档

- [混合搜索排序处理器]({{< relref "hybrid-ranker-processor.md" >}})：执行得分归一化和融合
- [混合搜索]({{< relref "/docs/integrations/ai/ai-api/hybrid-search.md" >}})：完整的混合搜索工作流和示例


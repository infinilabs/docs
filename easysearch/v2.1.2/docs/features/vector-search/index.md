---
title: "向量搜索"
date: 0001-01-01
description: "使用 k-NN 插件进行高维向量相似度搜索，支持稠密浮点向量与稀疏布尔向量。"
summary: "向量搜索 #  Easysearch 通过内置的 k-NN 插件提供高维向量相似度搜索能力，支持语义搜索、推荐系统、图像检索等 AI 应用场景。
核心能力 #     能力 说明     两种向量类型 稠密浮点向量（knn_dense_float_vector）和稀疏布尔向量（knn_sparse_bool_vector）   多种索引模型 lsh（局部敏感哈希，近似搜索）、permutation_lsh（置换 LSH）、sparse_indexed（倒排索引）、exact（精确搜索）   多种相似度 cosine（余弦）、l1（曼哈顿距离）、l2（欧氏距离）、jaccard、hamming   与全文搜索融合 向量字段与文本字段存储在同一索引，支持 Hybrid 混合检索   function_score 集成 向量相似度可作为 function_score 的评分函数    典型应用场景 #   语义搜索：文本通过 Embedding 模型转为向量，按语义相似度检索 RAG 检索增强生成：为大语言模型提供知识库检索能力 推荐系统：用户/商品特征向量的相似推荐 图像/多模态搜索：图像特征向量的相似检索 去重与异常检测：通过向量距离判断内容相似度  快速开始 #  // 1. 创建向量索引 PUT /my-vectors { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 768, &#34;model&#34;: &#34;lsh&#34;, &#34;similarity&#34;: &#34;cosine&#34;, &#34;L&#34;: 99, &#34;k&#34;: 1 } } } } } // 2."
---


# 向量搜索

Easysearch 通过内置的 k-NN 插件提供高维向量相似度搜索能力，支持语义搜索、推荐系统、图像检索等 AI 应用场景。

## 核心能力

| 能力 | 说明 |
|------|------|
| **两种向量类型** | 稠密浮点向量（`knn_dense_float_vector`）和稀疏布尔向量（`knn_sparse_bool_vector`） |
| **多种索引模型** | `lsh`（局部敏感哈希，近似搜索）、`permutation_lsh`（置换 LSH）、`sparse_indexed`（倒排索引）、`exact`（精确搜索） |
| **多种相似度** | `cosine`（余弦）、`l1`（曼哈顿距离）、`l2`（欧氏距离）、`jaccard`、`hamming` |
| **与全文搜索融合** | 向量字段与文本字段存储在同一索引，支持 Hybrid 混合检索 |
| **function_score 集成** | 向量相似度可作为 `function_score` 的评分函数 |

## 典型应用场景

- **语义搜索**：文本通过 Embedding 模型转为向量，按语义相似度检索
- **RAG 检索增强生成**：为大语言模型提供知识库检索能力
- **推荐系统**：用户/商品特征向量的相似推荐
- **图像/多模态搜索**：图像特征向量的相似检索
- **去重与异常检测**：通过向量距离判断内容相似度

## 快速开始

```json
// 1. 创建向量索引
PUT /my-vectors
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "embedding": {
        "type": "knn_dense_float_vector",
        "knn": {
          "dims": 768,
          "model": "lsh",
          "similarity": "cosine",
          "L": 99,
          "k": 1
        }
      }
    }
  }
}

// 2. 索引文档（向量由外部 Embedding 模型生成）
PUT /my-vectors/_doc/1
{
  "title": "Easysearch 向量搜索入门",
  "embedding": [0.12, -0.03, 0.88, ...]
}

// 3. 执行向量搜索
POST /my-vectors/_search
{
  "size": 10,
  "query": {
    "knn_nearest_neighbors": {
      "field": "embedding",
      "vec": { "values": [0.12, -0.03, 0.88, ...] },
      "model": "lsh",
      "similarity": "cosine",
      "candidates": 100
    }
  }
}
```

## 本章内容

| 页面 | 说明 |
|------|------|
| [向量搜索指南](./vector-search.md) | 从创建索引到查询的完整流程：Hybrid 检索、function_score、性能调优 |
| [向量字段建模](./vector-fields.md) | 多向量设计、维度选择、模型选型与写入策略 |
| [k-NN 查询 API](./knn_api.md) | knn_nearest_neighbors 查询参数完整参考 |
| [向量搜索与语义搜索](./vector-and-semantic-search.md) | 向量搜索、语义搜索、Hybrid 检索的定位区分与选择建议 |

## 相关资源

- [向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})
- [Embedding 服务集成]({{< relref "/docs/integrations/ai/embedding-service" >}})
- [向量工作流]({{< relref "/docs/integrations/ai/vector-workflow" >}})
- [AI API 集成]({{< relref "/docs/integrations/ai/ai-api/" >}})

---
title: "向量搜索与语义搜索"
date: 0001-01-01
description: "向量搜索、语义搜索、Hybrid 检索的定位区分与选择建议。"
summary: "向量搜索与语义搜索 #  向量搜索和语义搜索经常被混用，但它们面向的层次不同。理解它们的区别有助于选择正确的方案。
概念区分 #     概念 本质 Easysearch 中的对应     向量搜索 在高维空间中查找最近邻 knn_nearest_neighbors 查询   语义搜索 按&quot;含义&quot;而非&quot;关键词&quot;检索 文本 → Embedding → 向量搜索   Hybrid 检索 关键词 + 语义多路召回融合 bool（knn + match）   全文搜索 基于 BM25 的词频匹配 match、multi_match 等    关系：语义搜索 = Embedding 模型 + 向量搜索。向量搜索是底层能力，语义搜索是应用层方案。
什么时候用什么 #     场景 推荐方案 理由     用户查询关键词明确 全文搜索 BM25 对精确关键词匹配效果最好   用户用自然语言提问 语义搜索 Embedding 能捕获同义词和意图   既要关键词又要语义 Hybrid 检索 两路召回互补，综合效果最好   以图搜图、跨模态 向量搜索 不同模态映射到同一向量空间   RAG / 知识库问答 语义搜索 或 Hybrid 为 LLM 提供高质量上下文   推荐系统 向量搜索 用户/商品特征向量的近邻查找    端到端语义搜索流程 #  ┌──────────────┐ 用户查询 ──────▶ │ Embedding 模型│ ──────▶ 查询向量 └──────────────┘ │ ▼ ┌─────────────────┐ │ knn_nearest_ │ │ neighbors 查询 │ └─────────────────┘ │ ▼ 按语义相似度排序的结果  写入阶段：将文本通过 Embedding 模型转为向量，连同原始字段一起写入索引 查询阶段：将用户查询通过同一模型转为向量，使用 knn_nearest_neighbors 检索 融合阶段（可选）：将向量相似度得分与 BM25 得分加权融合  Embedding 模型的接入方式参见 Embedding 服务集成。"
---


# 向量搜索与语义搜索

向量搜索和语义搜索经常被混用，但它们面向的层次不同。理解它们的区别有助于选择正确的方案。

## 概念区分

| 概念 | 本质 | Easysearch 中的对应 |
|------|------|---------------------|
| **向量搜索** | 在高维空间中查找最近邻 | `knn_nearest_neighbors` 查询 |
| **语义搜索** | 按"含义"而非"关键词"检索 | 文本 → Embedding → 向量搜索 |
| **Hybrid 检索** | 关键词 + 语义多路召回融合 | `bool`（`knn` + `match`） |
| **全文搜索** | 基于 BM25 的词频匹配 | `match`、`multi_match` 等 |

**关系**：语义搜索 = Embedding 模型 + 向量搜索。向量搜索是底层能力，语义搜索是应用层方案。

## 什么时候用什么

| 场景 | 推荐方案 | 理由 |
|------|---------|------|
| 用户查询关键词明确 | 全文搜索 | BM25 对精确关键词匹配效果最好 |
| 用户用自然语言提问 | 语义搜索 | Embedding 能捕获同义词和意图 |
| 既要关键词又要语义 | Hybrid 检索 | 两路召回互补，综合效果最好 |
| 以图搜图、跨模态 | 向量搜索 | 不同模态映射到同一向量空间 |
| RAG / 知识库问答 | 语义搜索 或 Hybrid | 为 LLM 提供高质量上下文 |
| 推荐系统 | 向量搜索 | 用户/商品特征向量的近邻查找 |

## 端到端语义搜索流程

```
                  ┌──────────────┐
用户查询 ──────▶ │ Embedding 模型│ ──────▶ 查询向量
                  └──────────────┘           │
                                             ▼
                                    ┌─────────────────┐
                                    │ knn_nearest_     │
                                    │ neighbors 查询   │
                                    └─────────────────┘
                                             │
                                             ▼
                                    按语义相似度排序的结果
```

1. **写入阶段**：将文本通过 Embedding 模型转为向量，连同原始字段一起写入索引
2. **查询阶段**：将用户查询通过同一模型转为向量，使用 `knn_nearest_neighbors` 检索
3. **融合阶段**（可选）：将向量相似度得分与 BM25 得分加权融合

Embedding 模型的接入方式参见 [Embedding 服务集成]({{< relref "/docs/integrations/ai/embedding-service" >}})。

## Hybrid 检索示例

```json
POST /my-index/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "embedding",
            "vec": { "values": [0.12, -0.03, ...] },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 100
          }
        }
      ],
      "should": [
        {
          "match": {
            "content": {
              "query": "分布式搜索引擎",
              "boost": 2.0
            }
          }
        }
      ]
    }
  }
}
```

通过 `boost` 参数调节关键词得分与向量得分的权重比例。

## 相关资源

- [向量搜索指南](./vector-search.md)：完整的向量索引与查询操作
- [k-NN 查询 API](./knn_api.md)：查询参数完整参考
- [Embedding 服务集成]({{< relref "/docs/integrations/ai/embedding-service" >}})：接入外部 Embedding 模型
- [向量工作流]({{< relref "/docs/integrations/ai/vector-workflow" >}})：Embedding 生成与索引的工作流配置
- [AI API 集成]({{< relref "/docs/integrations/ai/ai-api/" >}})：Hybrid Search API 等高级功能


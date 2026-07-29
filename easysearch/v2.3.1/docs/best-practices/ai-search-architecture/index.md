---
title: "AI 搜索与向量检索架构实践"
date: 0001-01-01
description: "从召回到排序，如何用 Easysearch 搭建一条可落地的 AI/语义搜索链路。"
summary: "AI 搜索与向量检索架构实践 #  本页从整体架构的角度，讨论 Easysearch 在 AI/语义搜索系统中的位置：
 如何规划“全文 + 向量”的多路召回 如何利用 Easysearch 做向量检索与文档存储 如何与上游模型服务、下游应用/LLM 集成  详细的向量字段与 kNN API 参数，请参考 Mapping 与 Reference 中的相关章节，本页重点放在“怎么搭这条链路”。
1. 一条典型的 AI 搜索链路长什么样？ #  可以把 AI 搜索拆成几段：
 向量生产：由模型服务（自研或第三方）把文本/图片等转换成向量 向量与文档存储：在 Easysearch 里存放文档及其 Embedding 多路召回：  全文召回：BM25 等传统检索 向量召回：基于 kNN 的相似度检索   重排与融合：  按业务需要对多路召回结果做打分融合或模型重排   应用消费：  直接给用户展示 作为上下文提供给问答系统或大模型    Easysearch 重点负责第 2～4 段中的“存储 + 检索”能力。
2. 全文 + 向量：多路召回的常见拆法 #  结合 Easysearch，比较常见的做法有："
---


# AI 搜索与向量检索架构实践

本页从整体架构的角度，讨论 Easysearch 在 AI/语义搜索系统中的位置：

- 如何规划“全文 + 向量”的多路召回
- 如何利用 Easysearch 做向量检索与文档存储
- 如何与上游模型服务、下游应用/LLM 集成

详细的向量字段与 kNN API 参数，请参考 Mapping 与 Reference 中的相关章节，本页重点放在“怎么搭这条链路”。

## 1. 一条典型的 AI 搜索链路长什么样？

可以把 AI 搜索拆成几段：

1. **向量生产**：由模型服务（自研或第三方）把文本/图片等转换成向量
2. **向量与文档存储**：在 Easysearch 里存放文档及其 Embedding
3. **多路召回**：
   - 全文召回：BM25 等传统检索
   - 向量召回：基于 kNN 的相似度检索
4. **重排与融合**：
   - 按业务需要对多路召回结果做打分融合或模型重排
5. **应用消费**：
   - 直接给用户展示
   - 作为上下文提供给问答系统或大模型

Easysearch 重点负责第 2～4 段中的“存储 + 检索”能力。

## 2. 全文 + 向量：多路召回的常见拆法

结合 Easysearch，比较常见的做法有：

- **主路向量 + 辅助全文**：
  - 用向量检索作为主召回通道，保证语义相似
  - 用全文搜索做补充或过滤（例如强约束、精确匹配字段）
- **主路全文 + 向量重排**：
  - 第一阶段用全文搜索做大规模粗召回
  - 第二阶段对 TopN 文档用向量相似度做重排或融合评分
- **并行召回 + 融合**：
  - 一路全文、一向向量，并行检索
  - 合并结果时按权重或业务策略做打分组合

你可以根据场景（关键词检索 vs 自然语言问答）为不同入口选择不同策略。

## 3. Easysearch 在架构中的角色拆分

在一个典型部署中，可以把职责划分为：

- **模型服务**：负责生成与维护 Embedding（在线/离线）
- **数据接入层**：Logstash、自研服务等，负责把文档和向量写进 Easysearch
- **Easysearch 集群**：
  - 存储原始文档、结构化字段
  - 存储向量字段，并对外提供 kNN / Hybrid 搜索接口
- **应用层 / API 网关**：
  - 根据请求类型路由到不同检索策略
  - 负责把搜索结果转成前端/下游系统易消费的形态
- **（可选）LLM/问答服务**：
  - 调用 Easysearch 完成检索，再基于结果生成答案

Easysearch 不负责模型训练，但负责把“检索”这一环做到稳定可控。

## 4. 实战：Hybrid Search 配置示例

> **前置条件**：需要安装 knn 插件，参考 [插件安装]({{< relref "../deployment/install-guide#插件安装" >}})。

### 索引设计

Easysearch 使用 `knn_dense_float_vector` 类型存储密集型浮点向量，支持 LSH（近似搜索）和 Exact（精确搜索）两种模型。

```json
PUT /ai-knowledge-base
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": { "type": "text", "analyzer": "ik_max_word" },
      "content": { "type": "text", "analyzer": "ik_max_word" },
      "category": { "type": "keyword" },
      "embedding": {
        "type": "knn_dense_float_vector",
        "knn": {
          "dims": 768,
          "model": "lsh",
          "similarity": "cosine",
          "L": 99,
          "k": 1
        }
      },
      "created_at": { "type": "date" }
    }
  }
}
```

**参数说明**：
- `dims`：向量维度
- `model`：索引模型，`lsh` 为近似搜索，`exact` 为精确搜索
- `similarity`：相似度类型，支持 `cosine`、`l1`、`l2`
- `L`：哈希表数量，增大可提高召回率
- `k`：哈希函数数量，增大可提高精度

### Hybrid Search 查询

Easysearch 使用 `knn_nearest_neighbors` 查询进行向量检索，可与 `bool` 查询组合实现混合检索：

```json
POST /ai-knowledge-base/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "embedding",
            "vec": {
              "values": [0.12, -0.34, 0.56, ...]
            },
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
              "query": "如何优化搜索性能",
              "boost": 0.3
            }
          }
        }
      ],
      "filter": [
        { "term": { "category": "技术文档" } }
      ]
    }
  },
  "_source": ["title", "content", "category"]
}
```

**查询参数说明**：
- `field`：向量字段名
- `vec.values`：查询向量
- `model`：必须与索引时指定的模型一致
- `similarity`：相似度函数
- `candidates`：候选数量，增大可提高召回率

> **权重调参建议**：通过调整 `should` 子句的 `boost` 值控制全文与向量的相对权重。关键词检索场景可提高 BM25 权重，自然语言问答场景可减小或移除 `should` 中的全文查询。

## 5. 性能与成本权衡

| 维度       | 建议                                                         |
| ---------- | ------------------------------------------------------------ |
| 向量维度   | 50~768 维兼顾效果与性能；维度越高内存开销越大                |
| LSH 参数   | `L=99, k=1` 为常用起点，追求召回率可增大 `L`，追求精度可增大 `k` |
| 分片策略   | 向量索引建议单分片 5~10GB，避免过多小分片                     |
| 精确搜索   | 小数据量（<10 万文档）可用 `exact` 模型，无需配置额外参数     |
| 过滤优化   | 先用 `filter` 缩小范围再做向量检索，避免全量扫描             |

## 6. 和现有文档的关系

- 字段设计与向量存储：见 [向量字段]({{< relref "./data-modeling/vector-fields.md" >}})
- 向量工作流与写入方式：见 [向量工作流]({{< relref "../integrations/ai/vector-workflow.md" >}})
- RAG 与 LLM 集成细节：见 [RAG 与 LLM 集成]({{< relref "../integrations/ai/rag-and-llm.md" >}})
- Embedding 服务对接：见 [Embedding 服务接入]({{< relref "../integrations/ai/embedding-service.md" >}})



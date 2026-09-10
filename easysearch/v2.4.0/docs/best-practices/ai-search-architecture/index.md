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
  - 存储向量字段，并对外提供 kNN、复合查询和混合搜索接口
- **应用层 / API 网关**：
  - 根据请求类型路由到不同检索策略
  - 负责把搜索结果转成前端/下游系统易消费的形态
- **（可选）LLM/问答服务**：
  - 调用 Easysearch 完成检索，再基于结果生成答案

Easysearch 不负责模型训练，但负责把“检索”这一环做到稳定可控。

## 4. 实战：复合查询配置示例

以下示例使用 Easysearch 2.4.0 内置的原生 HNSW，不需要安装 k-NN 插件。已有旧插件索引继续使用旧字段和查询语法，
不要在同一字段上混用两套接口。

### 索引设计

Easysearch 使用 `dense_vector` 存储密集型浮点向量，并通过显式的 `index_options.type: hnsw` 创建原生 HNSW 索引。

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
        "type": "dense_vector",
        "dims": 768,
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "hnsw",
          "m": 16,
          "ef_construction": 100
        }
      },
      "created_at": { "type": "date" }
    }
  }
}
```

**参数说明**：

- `dims`：向量维度，必须与 Embedding 模型输出一致；
- `similarity`：相似度类型，可使用 `cosine`、`dot_product`、`l2_norm` 或 `max_inner_product`；
- `m`：图中每个节点保留的最大连接数；
- `ef_construction`：构图候选队列大小。

### 复合查询

原生 query-level `knn` 可以放入 `bool` 查询，与全文查询组合实现多路召回。这不是
[混合搜索]({{< relref "/docs/integrations/ai/hybrid-search.md" >}})；分数不可比、需要按排名融合时，使用搜索管道 RRF。

```json
POST /ai-knowledge-base/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "knn": {
            "field": "embedding",
            "query_vector": [0.12, -0.34, 0.56, ...],
            "k": 10,
            "num_candidates": 100,
            "filter": {
              "term": {
                "category": "技术文档"
              }
            }
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
      "minimum_should_match": 0
    }
  },
  "_source": ["title", "content", "category"]
}
```

**查询参数说明**：

- `field`：原生 `dense_vector` 字段名；
- `query_vector`：与 `dims` 一致的查询向量；
- `k`：每个分片保留的近邻候选数量；
- `num_candidates`：每个分片探索的候选数量，通常增大可提高 Recall，也会增加 CPU 和延迟；
- `filter`：在 HNSW 搜索中执行的预过滤条件。

> **权重调参建议**：通过调整子句的 `boost` 控制全文与向量分数的相对影响。BM25 与向量分数的分布不同，必须使用标注查询集验证
> Recall、NDCG 等指标，不能把 `boost` 直接解释为固定百分比。

## 5. 性能与成本权衡

| 维度       | 建议                                                         |
| ---------- | ------------------------------------------------------------ |
| 向量维度   | 由模型输出决定；在满足效果门槛后比较不同模型的写入、查询和存储成本 |
| HNSW 构图  | 从 `m=16`、`ef_construction=100` 起步，同时验证 Recall、写入吞吐和 store size |
| 查询候选   | `num_candidates` 从 `k` 的数倍起步，用真实查询集评估 Recall 与延迟 |
| 分片策略   | 用真实数据评估分片大小、恢复时间和查询并发；避免套用固定大小或创建过多小分片 |
| 过滤优化   | 将选择性条件放入 `knn.filter` 做预过滤，并验证过滤后的 Recall |

## 6. 和现有文档的关系

- 字段设计与向量存储：见 [向量字段]({{< relref "./data-modeling/vector-fields.md" >}})
- 原生 HNSW 完整合同：见 [原生 HNSW 搜索]({{< relref "/docs/features/vector-search/native-hnsw.md" >}})
- 向量工作流与写入方式：见 [向量工作流]({{< relref "../integrations/ai/vector-workflow.md" >}})
- RAG 与 LLM 集成细节：见 [RAG 与 LLM 集成]({{< relref "../integrations/ai/rag-and-llm.md" >}})
- Embedding 服务对接：见 [Embedding 服务接入]({{< relref "../integrations/ai/embedding-service.md" >}})


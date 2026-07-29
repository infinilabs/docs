---
title: "向量字段建模"
date: 0001-01-01
description: "向量字段设计策略：多向量、维度选择、模型选型与写入策略。"
summary: "向量字段建模 #  本页聚焦向量字段的设计策略——在已经理解字段类型和映射参数的基础上，如何为实际业务做出合理的字段规划。
 关于两种向量字段类型（knn_dense_float_vector / knn_sparse_bool_vector）的映射参数、数据格式、各索引模型的详细说明，请参阅 向量字段类型参考。
 文本字段 + 向量字段 #  一个典型的混合搜索文档同时包含文本字段和向量字段：
PUT /hybrid-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 768, &#34;model&#34;: &#34;lsh&#34;, &#34;similarity&#34;: &#34;cosine&#34;, &#34;L&#34;: 99, &#34;k&#34;: 1 } } } } } 这样同一份文档既支持 BM25 全文搜索，也支持 kNN 向量搜索，为 Hybrid 检索打基础。
多向量字段 #  为同一文档存储多个向量表示，例如标题和正文各一个 Embedding：
PUT /multi-vec-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;title_vec&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 768, &#34;model&#34;: &#34;lsh&#34;, &#34;similarity&#34;: &#34;cosine&#34;, &#34;L&#34;: 99, &#34;k&#34;: 1 } }, &#34;content_vec&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 768, &#34;model&#34;: &#34;lsh&#34;, &#34;similarity&#34;: &#34;cosine&#34;, &#34;L&#34;: 99, &#34;k&#34;: 1 } } } } }  注意：每个向量字段都消耗存储和索引资源。只为线上查询真正需要的维度建向量字段，避免无节制堆积。"
---


# 向量字段建模

本页聚焦向量字段的**设计策略**——在已经理解字段类型和映射参数的基础上，如何为实际业务做出合理的字段规划。

> 关于两种向量字段类型（`knn_dense_float_vector` / `knn_sparse_bool_vector`）的映射参数、数据格式、各索引模型的详细说明，请参阅 [向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。

## 文本字段 + 向量字段

一个典型的混合搜索文档同时包含文本字段和向量字段：

```json
PUT /hybrid-index
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" },
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
```

这样同一份文档既支持 BM25 全文搜索，也支持 kNN 向量搜索，为 [Hybrid 检索](./vector-search.md#hybrid-检索混合搜索)打基础。

## 多向量字段

为同一文档存储多个向量表示，例如标题和正文各一个 Embedding：

```json
PUT /multi-vec-index
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" },
      "title_vec": {
        "type": "knn_dense_float_vector",
        "knn": { "dims": 768, "model": "lsh", "similarity": "cosine", "L": 99, "k": 1 }
      },
      "content_vec": {
        "type": "knn_dense_float_vector",
        "knn": { "dims": 768, "model": "lsh", "similarity": "cosine", "L": 99, "k": 1 }
      }
    }
  }
}
```

> **注意**：每个向量字段都消耗存储和索引资源。只为线上查询真正需要的维度建向量字段，避免无节制堆积。

## 维度选择

- 维度由上游 Embedding 模型决定（常见 128、256、384、768、1024 等）
- 维度越高，存储开销越大、索引构建和查询越慢
- 在效果满足的前提下，**优先选择低维度模型**（如 256～512）
- 大维度向量多的索引建议按业务拆分

## 模型选择建议

| 场景 | 推荐字段类型 | 推荐模型 | 推荐相似度 |
|------|-------------|---------|-----------|
| 文本语义搜索 | `knn_dense_float_vector` | `lsh` | `cosine` |
| 图像特征检索 | `knn_dense_float_vector` | `lsh` | `cosine` 或 `l2` |
| 小数据集精确匹配 | `knn_dense_float_vector` | `exact` | `cosine` |
| 特征标签相似度 | `knn_sparse_bool_vector` | `lsh` 或 `sparse_indexed` | `jaccard` |
| 位串比较 | `knn_sparse_bool_vector` | `lsh` | `hamming` |

## 写入与更新策略

向量字段通常由外部模型计算得出，写入时要考虑：

- **同步写入**：写文档的同时调用 Embedding 模型生成向量，保证检索一致性
- **异步补齐**：先写文本文档，异步任务批量生成向量后 update，适合历史数据迁移

## 下一步

- [向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})：字段类型、映射参数、各索引模型详解
- [向量搜索指南](./vector-search.md)：完整的查询用法、Hybrid 检索、性能调优
- [k-NN 查询 API](./knn_api.md)：查询参数完整参考


---
title: "向量搜索"
date: 0001-01-01
description: "使用原生 HNSW、3.0 OSQ HNSW 或旧 k-NN 插件进行高维向量相似度搜索。"
summary: "向量搜索 #  Easysearch 2.4.0 内置基于 Lucene 的原生 HNSW 向量搜索。新建向量索引时，推荐使用 dense_vector mapping 和 knn 查询， 不需要安装 k-NN 插件。旧 k-NN 插件继续用于已有索引和兼容场景。
Easysearch 3.0 新增 OSQ HNSW 搜索，基于 Lucene 的 Optimized Scalar Quantization （优化的标量量化），用 1-bit 索引向量检索候选，再按配置以原精度向量重排。功能名为 Easysearch OSQ HNSW， 对外采用 ES 兼容参数 index_options.type: &quot;bbq_hnsw&quot;；省略该配置仍使用普通 HNSW。
核心能力 #     能力 原生 HNSW 说明     向量字段 单值 dense_vector；普通 HNSW 为 1–4096 维 float；3.0 OSQ HNSW 为 64–4096 维 float／bfloat16   索引 Lucene HNSW，支持 m 和 ef_construction；3."
---


# 向量搜索

Easysearch 2.4.0 内置基于 Lucene 的原生 HNSW 向量搜索。新建向量索引时，推荐使用 `dense_vector` mapping 和 `knn` 查询，
不需要安装 k-NN 插件。旧 k-NN 插件继续用于已有索引和兼容场景。

Easysearch 3.0 新增 [OSQ HNSW 搜索](./osq-hnsw.md)，基于 Lucene 的 Optimized Scalar Quantization
（优化的标量量化），用 1-bit 索引向量检索候选，再按配置以原精度向量重排。功能名为 **Easysearch OSQ HNSW**，
对外采用 ES 兼容参数 `index_options.type: "bbq_hnsw"`；省略该配置仍使用普通 HNSW。

## 核心能力

| 能力 | 原生 HNSW 说明 |
| --- | --- |
| 向量字段 | 单值 `dense_vector`；普通 HNSW 为 1–4096 维 `float`；3.0 OSQ HNSW 为 64–4096 维 `float`／`bfloat16` |
| 索引 | Lucene HNSW，支持 `m` 和 `ef_construction`；3.0 可显式使用 OSQ 量化及原精度重排 |
| 相似度 | `cosine`、`dot_product`、`l2_norm`、`max_inner_product` |
| 查询 | query-level `knn` 和协调全局 top-k 的顶层 `knn` |
| 过滤 | 在 HNSW 搜索中使用 `filter` 预过滤 |
| 复合查询 | query-level `knn` 可放入 `bool`、`dis_max`、`function_score` 等复合查询 |
| 客户端 | 已验证 Elasticsearch 8.19 Java 和 Python 官方客户端 |

## 快速开始

创建原生 HNSW 索引：

```json
PUT /my-vectors
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "embedding": {
        "type": "dense_vector",
        "dims": 4,
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "hnsw",
          "m": 16,
          "ef_construction": 100
        }
      }
    }
  }
}
```

写入一个向量：

```json
PUT /my-vectors/_doc/1?refresh=true
{
  "title": "Easysearch 向量搜索入门",
  "embedding": [1.0, 0.0, 0.0, 0.0]
}
```

执行 HNSW 搜索：

```json
POST /my-vectors/_search
{
  "size": 10,
  "query": {
    "knn": {
      "field": "embedding",
      "query_vector": [1.0, 0.0, 0.0, 0.0],
      "k": 10,
      "num_candidates": 100
    }
  }
}
```

完整参数、过滤、得分、调优、迁移和客户端示例见[原生 HNSW 搜索](./native-hnsw.md)，mapping 约束见
[dense_vector 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/dense-vector.md" >}})。

## 选择向量接口

原生 HNSW、OSQ HNSW 与旧 k-NN 插件的选择如下；旧插件与原生接口不能混用：

| 接口 | 字段类型 | 查询语法 | 适用场景 |
| --- | --- | --- | --- |
| [普通 HNSW](./native-hnsw.md) | `dense_vector`，`index_options.type: "hnsw"` | `knn` query、顶层 `knn` | 2.4.0 起内置，使用未量化浮点向量 |
| [OSQ HNSW](./osq-hnsw.md) | `dense_vector`，兼容 API 值 `index_options.type: "bbq_hnsw"` | `knn` query、顶层 `knn` | 3.0 新增，使用量化候选检索及可配置的原精度重排 |
| 旧 k-NN 插件 | `knn_dense_float_vector`、`knn_sparse_bool_vector` | `knn_nearest_neighbors` | 已有旧插件索引和兼容场景 |

旧插件需要单独安装，支持 `knn_dense_float_vector`、`knn_sparse_bool_vector`、LSH、exact 等既有接口。旧索引不会自动
转换为原生 HNSW；迁移时必须创建新索引并 Reindex。旧接口参考[旧 k-NN 查询 API](./knn_api.md)和
[旧 k-NN 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。

## 本章内容

| 页面                                                  | 说明                                                              |
| ----------------------------------------------------- | ----------------------------------------------------------------- |
| [原生 HNSW 搜索](./native-hnsw.md)                    | 2.4.0 `dense_vector`、query-level `knn` 和顶层 `knn`              |
| [OSQ HNSW 搜索](./osq-hnsw.md)                        | 3.0 映射、写入、候选重排、nested 和向量存储恢复 |
| [向量字段建模]({{< relref "/docs/best-practices/data-modeling/vector-fields.md" >}}) | 原生 HNSW 的维度、字段数量、写入、迁移和容量设计 |
| [旧插件向量搜索指南](./vector-search.md)              | 旧插件的复合查询、function_score 和性能调优                   |
| [旧插件向量字段建模](./vector-fields.md)              | 旧插件多向量设计、LSH/exact 模型选型与写入策略                    |
| [旧 k-NN 查询 API](./knn_api.md)                      | `knn_nearest_neighbors` 查询参数完整参考                           |
| [向量搜索与语义搜索](./vector-and-semantic-search.md) | 向量搜索、语义搜索、复合查询与混合搜索的定位区分               |

## 相关资源

- [dense_vector 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/dense-vector.md" >}})
- [旧 k-NN 字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})
- [Embedding 服务集成]({{< relref "/docs/integrations/ai/embedding-service" >}})
- [向量工作流]({{< relref "/docs/integrations/ai/vector-workflow" >}})
- [混合搜索]({{< relref "/docs/integrations/ai/hybrid-search.md" >}})：搜索管道 RRF（`hybrid_ranker_processor`）
- [AI API 集成]({{< relref "/docs/integrations/ai/_index.md" >}})

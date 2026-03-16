---
title: "向量工作流与 Hybrid 检索"
date: 0001-01-01
description: "围绕向量字段设计，从写入到检索的一整套工作流与 BM25+向量混合检索实践。"
summary: "向量工作流与 Hybrid 检索 #  本文完整介绍在 Easysearch 中使用向量检索的全流程——从索引设计、向量写入到混合检索策略。
相关指南（先读这些） #    Embedding 服务接入  向量检索功能  Hybrid Search API  索引设计 #  创建向量索引 #   从 1.11.1 版本开始，index.knn 参数已弃用，创建 knn 索引时无需配置。
 PUT knowledge_base { &#34;settings&#34;: { &#34;number_of_shards&#34;: 2, &#34;number_of_replicas&#34;: 1 }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;ik_max_word&#34;, &#34;search_analyzer&#34;: &#34;ik_smart&#34; }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;ik_max_word&#34; }, &#34;content_vector&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 768, &#34;model&#34;: &#34;lsh&#34;, &#34;similarity&#34;: &#34;cosine&#34;, &#34;L&#34;: 99, &#34;k&#34;: 1 } }, &#34;category&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;created_at&#34;: { &#34;type&#34;: &#34;date&#34; } } } } 向量字段参数 #     参数 说明     dims 向量维度，必须与 Embedding 模型输出维度一致   model 索引模型：lsh（近似搜索）或 exact（精确搜索）   similarity 相似度度量：cosine、l1、l2   L LSH 模型参数：哈希表数量，增大可提高召回率   k LSH 模型参数：哈希函数数量，增大可提高精度    向量写入 #  方式一：Ingest Pipeline 自动向量化 #  配置 Ingest Pipeline 在写入时自动调用 Embedding 服务："
---


# 向量工作流与 Hybrid 检索

本文完整介绍在 Easysearch 中使用向量检索的全流程——从索引设计、向量写入到混合检索策略。

## 相关指南（先读这些）

- [Embedding 服务接入]({{< relref "./embedding-service.md" >}})
- [向量检索功能]({{< relref "../../features/vector-search/_index.md" >}})
- [Hybrid Search API]({{< relref "./ai-api/hybrid-search.md" >}})

## 索引设计

### 创建向量索引

> 从 1.11.1 版本开始，`index.knn` 参数已弃用，创建 knn 索引时无需配置。

```
PUT knowledge_base
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "content": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "content_vector": {
        "type": "knn_dense_float_vector",
        "knn": {
          "dims": 768,
          "model": "lsh",
          "similarity": "cosine",
          "L": 99,
          "k": 1
        }
      },
      "category": { "type": "keyword" },
      "created_at": { "type": "date" }
    }
  }
}
```

### 向量字段参数

| 参数          | 说明                                                        |
| ------------- | ----------------------------------------------------------- |
| `dims`        | 向量维度，必须与 Embedding 模型输出维度一致                  |
| `model`       | 索引模型：`lsh`（近似搜索）或 `exact`（精确搜索）            |
| `similarity`  | 相似度度量：`cosine`、`l1`、`l2`                             |
| `L`           | LSH 模型参数：哈希表数量，增大可提高召回率                   |
| `k`           | LSH 模型参数：哈希函数数量，增大可提高精度                   |

## 向量写入

### 方式一：Ingest Pipeline 自动向量化

配置 Ingest Pipeline 在写入时自动调用 Embedding 服务：

```
PUT _ingest/pipeline/vectorize
{
  "processors": [
    {
      "text_embedding": {
        "model_id": "my-embedding-model",
        "field_map": {
          "content": "content_vector"
        }
      }
    }
  ]
}
```

写入时指定 pipeline：

```
POST knowledge_base/_doc?pipeline=vectorize
{
  "title": "Easysearch 集群配置",
  "content": "本文介绍 Easysearch 集群的配置方法和最佳实践...",
  "category": "tutorial"
}
```

### 方式二：应用侧预计算

在应用中先调用 Embedding 服务获取向量，再连同向量一起写入：

```
POST knowledge_base/_doc
{
  "title": "Easysearch 集群配置",
  "content": "本文介绍 Easysearch 集群的配置方法和最佳实践...",
  "content_vector": [0.12, -0.34, 0.56, ...],
  "category": "tutorial"
}
```

### 批量回填历史数据

对于已有的大量文本数据，可以用批处理脚本分批向量化并写入：

```python
from elasticsearch import Elasticsearch, helpers

es = Elasticsearch(["https://localhost:9200"], basic_auth=("admin", "pwd"), verify_certs=False)

# 分批读取现有文档
docs = helpers.scan(es, index="knowledge_base", query={"query": {"match_all": {}}})

actions = []
for doc in docs:
    vector = embedding_model.encode(doc["_source"]["content"])
    actions.append({
        "_op_type": "update",
        "_index": "knowledge_base",
        "_id": doc["_id"],
        "doc": {"content_vector": vector.tolist()}
    })
    if len(actions) >= 100:
        helpers.bulk(es, actions)
        actions = []
```

## Hybrid 检索策略

混合检索将 BM25 全文搜索和向量 kNN 搜索的结果融合，兼顾精确匹配和语义理解：

### 基础 Hybrid 查询

```
POST knowledge_base/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "content_vector",
            "vec": {
              "values": [0.12, -0.34, ...]
            },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 50
          }
        }
      ],
      "should": [
        {
          "match": {
            "content": {
              "query": "集群健康检查",
              "boost": 0.3
            }
          }
        }
      ]
    }
  }
}
```

### 权重调优

| BM25 权重 | kNN 权重 | 效果                                     |
| --------- | -------- | ---------------------------------------- |
| 0.7       | 0.3      | 偏向精确匹配，适合关键词明确的查询       |
| 0.3       | 0.7      | 偏向语义理解，适合自然语言问句           |
| 0.5       | 0.5      | 平衡模式，通用场景推荐起点               |

> **调参建议**：使用一组标注好的查询-文档对来评估不同权重组合的检索效果（Precision@K、NDCG 等），找到最适合业务的权重配比。



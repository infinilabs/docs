---
title: "向量工作流与复合检索"
date: 0001-01-01
description: "围绕向量字段设计，从写入到检索的一整套工作流与 BM25 + 向量复合查询实践。"
summary: "向量工作流与复合检索 #  本文完整介绍在 Easysearch 中使用向量检索的全流程——从索引设计、向量写入到复合查询。
以下 mapping 和查询默认使用 Easysearch 2.4.0 原生 HNSW，不需要安装 k-NN 插件。旧插件索引的字段与查询语法不同， 参考 旧插件向量搜索指南。
相关指南（先读这些） #    Embedding 服务接入  向量检索功能  原生 HNSW 搜索  混合搜索：搜索管道 RRF（hybrid_ranker_processor）  索引设计 #  创建向量索引 #  PUT knowledge_base { &#34;settings&#34;: { &#34;number_of_shards&#34;: 2, &#34;number_of_replicas&#34;: 1 }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;ik_max_word&#34;, &#34;search_analyzer&#34;: &#34;ik_smart&#34; }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;ik_max_word&#34; }, &#34;content_vector&#34;: { &#34;type&#34;: &#34;dense_vector&#34;, &#34;dims&#34;: 768, &#34;index&#34;: true, &#34;similarity&#34;: &#34;cosine&#34; }, &#34;category&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;created_at&#34;: { &#34;type&#34;: &#34;date&#34; } } } }  重要：省略 index_options 后仍会使用 HNSW。 Easysearch 2."
---


# 向量工作流与复合检索

本文完整介绍在 Easysearch 中使用向量检索的全流程——从索引设计、向量写入到复合查询。

以下 mapping 和查询默认使用 Easysearch 2.4.0 原生 HNSW，不需要安装 k-NN 插件。旧插件索引的字段与查询语法不同，
参考[旧插件向量搜索指南]({{< relref "/docs/features/vector-search/vector-search.md" >}})。

## 相关指南（先读这些）

- [Embedding 服务接入]({{< relref "./embedding-service.md" >}})
- [向量检索功能]({{< relref "../../features/vector-search/_index.md" >}})
- [原生 HNSW 搜索]({{< relref "../../features/vector-search/native-hnsw.md" >}})
- [混合搜索]({{< relref "./hybrid-search.md" >}})：搜索管道 RRF（`hybrid_ranker_processor`）

## 索引设计

### 创建向量索引

```json
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
        "type": "dense_vector",
        "dims": 768,
        "index": true,
        "similarity": "cosine"
      },
      "category": { "type": "keyword" },
      "created_at": { "type": "date" }
    }
  }
}
```

> **重要：省略 `index_options` 后仍会使用 HNSW。** Easysearch 2.4.0 默认使用并回显
> `hnsw(m=16, ef_construction=100)`；需要调整构图参数时再显式配置 `index_options`。

### 向量字段参数

| 参数 | 说明 |
| --- | --- |
| `dims` | 向量维度，必须与 Embedding 模型输出维度一致 |
| `index` | 2.4.0 原生 `dense_vector` 必须显式设置为 `true` |
| `similarity` | `cosine`、`dot_product`、`l2_norm` 或 `max_inner_product` |
| `index_options` | 可省略；默认为 `hnsw(m=16, ef_construction=100)` |
| `m` | 显式设置 `index_options` 时可调；HNSW 图中每个节点保留的最大连接数 |
| `ef_construction` | 显式设置 `index_options` 时可调；构图候选队列大小 |

## 向量写入

### 方式一：Ingest Pipeline 自动向量化

该方式需要安装 AI 插件。配置 Ingest Pipeline 在写入时自动调用 Embedding 服务；`dims` 必须与目标 `dense_vector` mapping 和模型
输出一致：

```json
PUT _ingest/pipeline/vectorize
{
  "processors": [
        {
          "text_embedding": {
            "url": "https://api.openai.com/v1/embeddings",
            "vendor": "openai",
            "api_key": "<api_key>",
            "text_field": "content",
            "vector_field": "content_vector",
            "model_id": "text-embedding-3-small",
            "dims": 768,
            "ignore_missing": false,
            "ignore_failure": false
          }
    }
  ]
}
```

写入时指定 pipeline：

```json
POST knowledge_base/_doc?pipeline=vectorize
{
  "title": "Easysearch 集群配置",
  "content": "本文介绍 Easysearch 集群的配置方法和最佳实践...",
  "category": "tutorial"
}
```

### 方式二：应用侧预计算

在应用中先调用 Embedding 服务获取向量，再连同向量一起写入：

```json
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

## 复合查询策略

query-level `knn` 可以与 `match` 放进同一个 `bool` 查询，兼顾精确匹配和语义理解。这是复合查询，不是
[混合搜索]({{< relref "./hybrid-search.md" >}})。文档中的 Hybrid 只指搜索管道 RRF。

### 基础复合查询

```json
POST knowledge_base/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "knn": {
            "field": "content_vector",
            "query_vector": [0.12, -0.34, ...],
            "k": 10,
            "num_candidates": 50
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

可以分别设置 `knn` 和 `match` 子句的 `boost`，调整两类分数对最终排序的相对影响。BM25 与向量分数不在同一分布，`boost` 不是
百分比权重。应使用一组标注好的查询-文档对评估 Precision@K、Recall@K、NDCG 等指标，再确定业务所需的参数。


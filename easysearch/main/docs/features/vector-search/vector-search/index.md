---
title: "向量搜索指南"
date: 0001-01-01
description: "从向量字段配置到 kNN 查询、Hybrid 检索的完整指南。"
summary: "向量搜索指南 #  本页介绍如何在 Easysearch 中使用向量搜索——从创建索引到执行查询的完整流程。
前提条件 #  向量搜索需要 k-NN 插件。安装方法参见 插件安装。
 注意：从 1.11.1 版本起，创建 k-NN 索引时不再需要配置 index.knn 参数。
 步骤一：创建向量索引 #  稠密浮点向量（最常用） #  适用于文本 Embedding、图像特征等场景：
PUT /my-vectors { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 768, &#34;model&#34;: &#34;lsh&#34;, &#34;similarity&#34;: &#34;cosine&#34;, &#34;L&#34;: 99, &#34;k&#34;: 1 } } } } }  映射参数、各索引模型和相似度函数的完整说明，请参阅 向量字段类型参考。
 步骤二：索引文档 #  向量通常由外部 Embedding 模型生成，写入时附带向量数据："
---


# 向量搜索指南

本页介绍如何在 Easysearch 中使用向量搜索——从创建索引到执行查询的完整流程。

## 前提条件

向量搜索需要 k-NN 插件。安装方法参见[插件安装]({{< relref "/docs/deployment/install-guide/_index.md" >}})。

> **注意**：从 1.11.1 版本起，创建 k-NN 索引时不再需要配置 `index.knn` 参数。

## 步骤一：创建向量索引

### 稠密浮点向量（最常用）

适用于文本 Embedding、图像特征等场景：

```json
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
```

> 映射参数、各索引模型和相似度函数的完整说明，请参阅 [向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。

## 步骤二：索引文档

向量通常由外部 Embedding 模型生成，写入时附带向量数据：

```json
PUT /my-vectors/_doc/1
{
  "title": "Easysearch 分布式搜索引擎",
  "embedding": [0.12, -0.03, 0.88, 0.45, ...]
}

PUT /my-vectors/_doc/2
{
  "title": "向量相似度搜索原理",
  "embedding": [-0.05, 0.22, 0.67, 0.31, ...]
}
```

> **提示**：向量维度必须与映射中 `dims` 参数一致，否则写入会报错。

## 步骤三：执行向量搜索

### 近似搜索（LSH）

LSH 模型提供高效的近似最近邻搜索，适用于大规模数据集：

```json
POST /my-vectors/_search
{
  "size": 10,
  "query": {
    "knn_nearest_neighbors": {
      "field": "embedding",
      "vec": {
        "values": [0.12, -0.03, 0.88, 0.45, ...]
      },
      "model": "lsh",
      "similarity": "cosine",
      "candidates": 100
    }
  }
}
```

#### 查询参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `field` | String | 向量字段名 |
| `vec.values` | Float[] | 查询向量 |
| `model` | String | 必须与创建索引时一致 |
| `similarity` | String | 相似度函数 |
| `candidates` | Integer | 候选数量，越大越准但越慢。推荐值：`size` 的 5～10 倍 |

### 精确搜索（Exact）

精确模型计算查询向量与所有文档的精确相似度，适用于小数据集或对精度要求极高的场景：

```json
POST /my-vectors/_search
{
  "size": 10,
  "query": {
    "knn_nearest_neighbors": {
      "field": "embedding",
      "vec": {
        "values": [0.12, -0.03, 0.88, 0.45, ...]
      },
      "model": "exact",
      "similarity": "cosine"
    }
  }
}
```

> ⚠️ 精确搜索的时间复杂度为 O(n²)，文档数量较大时性能会显著下降。

### 引用已索引向量

可以用索引中已有文档的向量作为查询向量，无需在请求中传递完整向量值：

```json
POST /my-vectors/_search
{
  "size": 10,
  "query": {
    "knn_nearest_neighbors": {
      "field": "embedding",
      "vec": {
        "index": "my-vectors",
        "id": "1",
        "field": "embedding"
      },
      "model": "lsh",
      "similarity": "cosine",
      "candidates": 100
    }
  }
}
```

## 带过滤的向量搜索

向量搜索可以与布尔查询组合，实现"语义相似 + 结构化过滤"：

```json
POST /my-vectors/_search
{
  "size": 5,
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "embedding",
            "vec": { "values": [0.12, -0.03, ...] },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 50
          }
        }
      ],
      "filter": [
        { "term": { "category": "tech" } },
        { "range": { "created_at": { "gte": "now-30d/d" } } }
      ]
    }
  }
}
```

> **注意**：过滤在向量召回之后执行（post-filtering），先通过 kNN 召回候选集，再用 filter 筛选。

## Hybrid 检索（混合搜索）

将向量相似度与 BM25 全文搜索融合，同时利用关键词匹配和语义理解：

```json
POST /my-vectors/_search
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
            "title": {
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

工作原理：

1. `knn_nearest_neighbors` 找出语义相似的文档（向量相似度得分）
2. `match` 找出关键词匹配的文档（BM25 得分）
3. Easysearch 将两路得分融合，输出综合排序

> 可以通过 `boost` 参数调节两路得分的权重比例。

## 使用 function_score 集成向量评分

向量相似度也可以作为 `function_score` 中的评分函数，与其他评分逻辑灵活组合：

```json
POST /my-vectors/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": { "title": "搜索引擎" }
      },
      "functions": [
        {
          "knn_nearest_neighbors": {
            "field": "embedding",
            "vec": { "values": [0.12, -0.03, ...] },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 100
          },
          "weight": 2
        }
      ],
      "boost_mode": "multiply"
    }
  }
}
```


## 性能调优

### LSH 参数调优

- **`L`（哈希表数量）**：增大提高召回率，但增加内存和索引时间。建议从 50～100 开始
- **`k`（哈希函数数量）**：增大提高精度但可能降低召回率。建议从 1～3 开始
- **`candidates`**：查询时的候选数量，一般设为 `size` 的 5～10 倍

### 容量规划

- 维度越高，单条文档的向量存储开销越大
- 在满足效果的前提下，**尽量使用较低维度的模型**（如 256～512 维）
- 避免一个索引里堆积过多向量字段，必要时按业务拆分索引

### 写入策略

- **同步写入**：写文档的同时调用 Embedding 模型生成向量，保证检索一致性
- **异步补齐**：先写文本文档，异步任务批量生成向量后 update，适合历史数据迁移

## 下一步

- [向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})：字段类型、映射参数、各索引模型详解
- [向量字段建模](./vector-fields.md)：多向量设计、维度选择、模型选型策略
- [k-NN 查询 API](./knn_api.md)：完整的 API 参数参考
- [Embedding 服务集成]({{< relref "/docs/integrations/ai/embedding-service" >}})：接入外部 Embedding 模型
- [AI API 集成]({{< relref "/docs/integrations/ai/ai-api/" >}})：Hybrid Search API 等高级功能


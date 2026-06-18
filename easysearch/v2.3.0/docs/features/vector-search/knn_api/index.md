---
title: "k-NN 查询 API"
date: 0001-01-01
description: "knn_nearest_neighbors 查询的完整参数参考：所有模型、相似度函数和查询选项。"
summary: "k-NN 查询 API #  先决条件 #  要运行 k-NN 搜索，必须安装 knn 插件，参考 插件安装。
 注意：从 1.11.1 版本起，创建 k-NN 索引时不再需要配置 index.knn 参数。
  向量字段的映射参数、各索引模型和相似度函数的详细说明，请参阅 向量字段类型参考。
 查询语法 #  所有向量搜索都通过 knn_nearest_neighbors 查询完成：
GET /&lt;index&gt;/_search { &#34;query&#34;: { &#34;knn_nearest_neighbors&#34;: { &#34;field&#34;: &#34;&lt;vector_field&gt;&#34;, &#34;vec&#34;: { ... }, &#34;model&#34;: &#34;&lt;model&gt;&#34;, &#34;similarity&#34;: &#34;&lt;similarity&gt;&#34;, &#34;candidates&#34;: &lt;n&gt; } } } 查询参数 #     参数 类型 必填 说明     field String ✅ 向量字段名称   vec Object ✅ 查询向量，支持三种格式（见下方）   model String ✅ 索引模型：lsh、exact、permutation_lsh、sparse_indexed   similarity String ✅ 相似度函数，必须与映射中的配置一致   candidates Integer 否 近似搜索的候选数量。仅用于 lsh/permutation_lsh/sparse_indexed 模型。越大越精准但越慢，推荐 size 的 5～10 倍   probes Integer 否 仅用于 l2 相似度的 LSH 查询：探测数量    查询向量格式 #  1."
---


# k-NN 查询 API

## 先决条件

要运行 k-NN 搜索，必须安装 `knn` 插件，参考[插件安装]({{< relref "/docs/deployment/install-guide/_index.md" >}})。

> **注意**：从 1.11.1 版本起，创建 k-NN 索引时不再需要配置 `index.knn` 参数。

> 向量字段的映射参数、各索引模型和相似度函数的详细说明，请参阅 [向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。

## 查询语法

所有向量搜索都通过 `knn_nearest_neighbors` 查询完成：

```json
GET /<index>/_search
{
  "query": {
    "knn_nearest_neighbors": {
      "field": "<vector_field>",
      "vec": { ... },
      "model": "<model>",
      "similarity": "<similarity>",
      "candidates": <n>
    }
  }
}
```

## 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `field` | String | ✅ | 向量字段名称 |
| `vec` | Object | ✅ | 查询向量，支持三种格式（见下方） |
| `model` | String | ✅ | 索引模型：`lsh`、`exact`、`permutation_lsh`、`sparse_indexed` |
| `similarity` | String | ✅ | 相似度函数，必须与映射中的配置一致 |
| `candidates` | Integer | 否 | 近似搜索的候选数量。仅用于 `lsh`/`permutation_lsh`/`sparse_indexed` 模型。越大越精准但越慢，推荐 `size` 的 5～10 倍 |
| `probes` | Integer | 否 | 仅用于 `l2` 相似度的 LSH 查询：探测数量 |

## 查询向量格式

### 1. 直接提供值（最常用）

```json
"vec": {
  "values": [0.12, -0.03, 0.88, 0.45, ...]
}
```

用于稠密浮点向量（`knn_dense_float_vector`），数组长度必须与映射中的 `dims` 一致。

### 2. 引用已索引向量

```json
"vec": {
  "index": "my-vectors",
  "id": "doc-1",
  "field": "embedding"
}
```

用已存在文档的向量作为查询向量，避免在请求中传递完整向量。参数说明：

| 参数 | 说明 |
|------|------|
| `index` | 源文档所在的索引名 |
| `id` | 源文档 ID |
| `field` | 源文档中的向量字段名 |

### 3. 稀疏布尔向量

```json
"vec": {
  "values": [[0, 3, 5, 12, 88], 1000]
}
```

用于稀疏布尔向量（`knn_sparse_bool_vector`），格式为 `[[true_indices], total_dims]`。

---

## LSH 近似搜索

最常用的查询方式，通过局部敏感哈希实现高效近似最近邻搜索。

### 稠密浮点向量 + LSH

以 GloVe 词向量为例，查找与 "bread" 最相似的词：

```json
GET /glove-vectors/_search
{
  "size": 5,
  "_source": "word",
  "query": {
    "knn_nearest_neighbors": {
      "field": "my_vec",
      "vec": {
        "values": [-0.37436, -0.11959, -0.87609, -1.1217, 1.2788, 0.48323, ...]
      },
      "model": "lsh",
      "similarity": "cosine",
      "candidates": 50
    }
  }
}
```

#### 响应示例

```json
{
  "took": 214,
  "hits": {
    "total": { "value": 400, "relation": "eq" },
    "max_score": 2,
    "hits": [
      {
        "_index": "glove-vectors",
        "_id": "WYPvCYkBkPNAx5w6LJ6H",
        "_score": 2,
        "_source": { "word": "bread" }
      },
      {
        "_index": "glove-vectors",
        "_id": "PYPvCYkBkPNAx5w6Mbui",
        "_score": 1.8483887,
        "_source": { "word": "baked" }
      },
      {
        "_index": "glove-vectors",
        "_id": "RoPvCYkBkPNAx5w6NsYl",
        "_score": 1.8451341,
        "_source": { "word": "toast" }
      },
      {
        "_index": "glove-vectors",
        "_id": "o4PvCYkBkPNAx5w6LKCI",
        "_score": 1.84022,
        "_source": { "word": "butter" }
      },
      {
        "_index": "glove-vectors",
        "_id": "koPvCYkBkPNAx5w6LKeK",
        "_score": 1.8374994,
        "_source": { "word": "soup" }
      }
    ]
  }
}
```

> 使用 `cosine` 相似度时，得分范围为 [0, 2]。完全相同的向量得分为 2。

### 稀疏布尔向量 + LSH

#### 索引文档

```json
PUT /sparse-features/_doc/1
{
  "label": "文档A",
  "features": [[0, 3, 5, 12, 88, 200, 456], 1000]
}
```

#### 查询示例

```json
GET /sparse-features/_search
{
  "size": 5,
  "query": {
    "knn_nearest_neighbors": {
      "field": "features",
      "vec": {
        "values": [[0, 5, 12, 100, 456], 1000]
      },
      "model": "lsh",
      "similarity": "jaccard",
      "candidates": 50
    }
  }
}
```

---

## 精确搜索

精确模型计算查询向量与所有索引向量的精确相似度，不使用任何近似索引结构。映射时不需要指定 `model`，详见[向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。

### 查询

```json
GET /exact-index/_search
{
  "query": {
    "knn_nearest_neighbors": {
      "field": "my_vec",
      "vec": {
        "values": [0.1, 0.2, 0.3, ...]
      },
      "model": "exact",
      "similarity": "cosine"
    }
  }
}
```

> ⚠️ 精确搜索的时间复杂度为 O(n²)，适用于文档数量较少的场景。

精确搜索支持的相似度：

| 向量类型 | 可用相似度 |
|---------|-----------|
| `knn_dense_float_vector` | `cosine`、`l1`、`l2` |
| `knn_sparse_bool_vector` | `jaccard`、`hamming` |

---

## 引用已索引向量

使用索引中已有文档的向量作为查询向量，适用于"查找相似文档"的场景：

```json
GET /my-vectors/_search
{
  "size": 10,
  "query": {
    "knn_nearest_neighbors": {
      "field": "embedding",
      "vec": {
        "index": "my-vectors",
        "id": "doc-1",
        "field": "embedding"
      },
      "model": "lsh",
      "similarity": "cosine",
      "candidates": 100
    }
  }
}
```

> 源文档可以在同一索引或不同索引中，只要向量类型和维度匹配即可。

---

## sparse_indexed 模型

`sparse_indexed` 模型使用倒排索引结构，专为稀疏布尔向量设计。映射配置详见[向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。

### 查询

```json
GET /sparse-idx/_search
{
  "query": {
    "knn_nearest_neighbors": {
      "field": "features",
      "vec": {
        "values": [[1, 10, 50, 200, 1000], 5000]
      },
      "model": "sparse_indexed",
      "similarity": "jaccard",
      "candidates": 50
    }
  }
}
```

---

## permutation_lsh 模型

`permutation_lsh` 是基于排列的局部敏感哈希，仅适用于稠密浮点向量。映射配置详见[向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。

### 查询

```json
GET /perm-index/_search
{
  "query": {
    "knn_nearest_neighbors": {
      "field": "embedding",
      "vec": {
        "values": [0.1, 0.2, ...]
      },
      "model": "permutation_lsh",
      "similarity": "cosine",
      "candidates": 100
    }
  }
}
```

---

## 与 bool 查询组合

向量搜索可以嵌入 `bool` 查询，与过滤条件组合：

```json
GET /my-vectors/_search
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
      "filter": [
        { "term": { "category": "tech" } }
      ]
    }
  }
}
```

## 与 function_score 集成

向量相似度可作为 `function_score` 的评分函数：

```json
GET /my-vectors/_search
{
  "query": {
    "function_score": {
      "query": { "match": { "title": "搜索引擎" } },
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

---

## 相关页面

- [向量字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})：字段类型、映射参数、各索引模型详解
- [向量搜索指南](./vector-search.md)：完整的使用流程与 Hybrid 检索
- [向量字段建模](./vector-fields.md)：多向量设计、维度选择、模型选型策略


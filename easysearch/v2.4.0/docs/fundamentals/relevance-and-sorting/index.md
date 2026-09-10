---
title: "相关性与排序"
date: 0001-01-01
description: "理解 _score 评分的计算原理（BM25）、默认参数、自定义排序、多字段排序与调试评分。"
summary: "相关性与排序 #  在 Easysearch 中，搜索结果默认按**相关性（relevance）**降序排列。理解相关性如何计算、如何自定义排序，是用好全文搜索的关键。
什么是相关性？ #  每个文档都有一个 _score 字段，表示它与查询的匹配程度。评分越高，相关性越高。
_score 的计算基于 BM25 算法（Okapi BM25），它是经典 TF/IDF 的改进版本。BM25 使用三个核心因素来衡量相关性：
词频（Term Frequency, TF） #  检索词在文档字段中出现的频率。出现 5 次比出现 1 次的权重更高。BM25 对词频做了饱和处理——频率增长到一定程度后增益递减，不像经典 TF/IDF 会无限增长：
$$tf_{BM25} = \frac{freq \cdot (k_1 + 1)}{freq + k_1 \cdot (1 - b + b \cdot \frac{dl}{avgdl})}$$
逆向文档频率（Inverse Document Frequency, IDF） #  检索词在所有文档中出现的频率。越常见的词（如&quot;的&quot;&ldquo;和&rdquo;），权重越低；越罕见的词权重越高。
$$idf(t) = \ln\left(1 + \frac{N - df + 0.5}{df + 0.5}\right)$$
其中 $N$ 是文档总数，$df$ 是包含该词的文档数。"
---


# 相关性与排序

在 Easysearch 中，搜索结果默认按**相关性（relevance）**降序排列。理解相关性如何计算、如何自定义排序，是用好全文搜索的关键。

## 什么是相关性？

每个文档都有一个 `_score` 字段，表示它与查询的匹配程度。评分越高，相关性越高。

`_score` 的计算基于 **BM25** 算法（Okapi BM25），它是经典 TF/IDF 的改进版本。BM25 使用三个核心因素来衡量相关性：

### 词频（Term Frequency, TF）

检索词在文档字段中出现的频率。出现 5 次比出现 1 次的权重更高。BM25 对词频做了**饱和处理**——频率增长到一定程度后增益递减，不像经典 TF/IDF 会无限增长：

$$tf_{BM25} = \frac{freq \cdot (k_1 + 1)}{freq + k_1 \cdot (1 - b + b \cdot \frac{dl}{avgdl})}$$

### 逆向文档频率（Inverse Document Frequency, IDF）

检索词在所有文档中出现的频率。越常见的词（如"的""和"），权重越低；越罕见的词权重越高。

$$idf(t) = \ln\left(1 + \frac{N - df + 0.5}{df + 0.5}\right)$$

其中 $N$ 是文档总数，$df$ 是包含该词的文档数。

### 字段长度归一化（Field-length Norm）

字段越短，权重越高。检索词出现在 `title`（短字段）比出现在 `content`（长字段）更有意义。在 BM25 中通过参数 $b$ 控制长度归一化的影响程度。

### BM25 默认参数

| 参数 | 默认值 | 作用 |
|------|--------|------|
| **k1** | 1.2 | 控制词频饱和速度。值越大，高频词获得的额外分数越多 |
| **b** | 0.75 | 控制字段长度归一化的强度。0 = 不归一化，1 = 完全按比例归一化 |
| **discount_overlaps** | true | 位置重叠的词条（如同义词）不计入长度 |

可以在 mapping 中为特定字段自定义 BM25 参数：

```json
PUT /my_index
{
  "settings": {
    "similarity": {
      "custom_bm25": {
        "type": "BM25",
        "k1": 1.5,
        "b": 0.3
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "similarity": "custom_bm25"
      }
    }
  }
}
```

这三个因素结合起来，决定了一个词在特定文档中的**权重**，最终汇总为 `_score`。

## 理解评分：explain

使用 `explain` 参数可以查看评分的详细计算过程：

```json
GET /my_index/_search?explain=true
{
  "query": {
    "match": { "content": "search" }
  }
}
```

返回中的 `_explanation` 会详细列出 TF、IDF、field norm 等各因子的值。

也可以查看**某个文档为什么不匹配**：

```json
GET /my_index/_explain/123
{
  "query": {
    "match": { "content": "search" }
  }
}
```

> ⚠️ `explain` 开销很大，仅用于调试，不要在生产环境使用。

## 按字段排序

不是所有场景都需要相关性排序。可以使用 `sort` 参数按任意字段排序：

```json
GET /_search
{
  "query": {
    "bool": {
      "filter": { "term": { "status": "published" } }
    }
  },
  "sort": { "date": { "order": "desc" } }
}
```

使用 `sort` 后：
- `_score` 不会被计算（显示为 `null`），节省性能
- 结果中会有 `sort` 字段显示排序值
- 如果还需要 `_score`，设置 `"track_scores": true`

### 多级排序

可以按多个字段依次排序：

```json
GET /_search
{
  "sort": [
    { "date":   { "order": "desc" } },
    { "_score": { "order": "desc" } }
  ]
}
```

先按日期降序，日期相同时按相关性降序。

### 多值字段排序

对于多值字段（如数组），可以用 `mode` 指定取哪个值排序：

```json
"sort": {
  "prices": {
    "order": "asc",
    "mode": "min"
  }
}
```

可选模式：`min`、`max`、`avg`、`sum`。

## 字符串排序与多字段

`text` 类型字段会被分析成多个词条，直接用于排序可能得到意外结果。推荐做法是使用 **multi-fields**：

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

搜索用 `title`（全文匹配），排序用 `title.keyword`（精确值）：

```json
GET /_search
{
  "query": { "match": { "title": "search engine" } },
  "sort": "title.keyword"
}
```

## 小结

| 概念 | 要点 |
|------|------|
| `_score` | 相关性评分，基于 BM25（k1=1.2, b=0.75） |
| TF | 词频越高，权重越高 |
| IDF | 文档频率越高，权重越低（常见词权重低） |
| Field norm | 字段越短，权重越高 |
| `sort` | 按字段排序，跳过评分计算 |
| Multi-fields | `text` + `keyword` 子字段，分别用于搜索和排序 |
| `explain` | 调试评分的利器，仅用于开发环境 |

下一步可以继续阅读：

- [全文搜索](../features/fulltext-search/)：搜索 DSL 的完整使用
- [映射与分析入门](./mapping-analysis-intro.md)：分析器与字段类型
- [分页与排序](../features/fulltext-search/pagination-and-sorting.md)：深分页优化

### 最佳实践

- [查询调优与慢查询排查]({{< relref "/docs/best-practices/query-tuning.md" >}})：相关性调优、评分影响因素
- [数据建模]({{< relref "/docs/best-practices/data-modeling/" >}})：多字段策略对排序的影响


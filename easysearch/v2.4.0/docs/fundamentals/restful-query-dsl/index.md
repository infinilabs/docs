---
title: "RESTful 与 Query DSL"
date: 0001-01-01
summary: "RESTful 与 Query DSL #  Easysearch 通过 RESTful API 提供所有功能，使用 JSON 格式的 Query DSL 描述查询逻辑。理解这两个基础概念，是使用 Easysearch 的第一步。
RESTful API 约定 #  Easysearch 的所有操作都通过 HTTP 请求完成，遵循 RESTful 风格：
   HTTP 方法 含义 示例     GET 读取 GET /my-index/_search   PUT 创建或全量更新 PUT /my-index   POST 创建或部分更新 POST /my-index/_doc   DELETE 删除 DELETE /my-index    请求结构 #  一个典型的 Easysearch 请求由三部分组成："
---


# RESTful 与 Query DSL

Easysearch 通过 **RESTful API** 提供所有功能，使用 **JSON 格式的 Query DSL** 描述查询逻辑。理解这两个基础概念，是使用 Easysearch 的第一步。

## RESTful API 约定

Easysearch 的所有操作都通过 HTTP 请求完成，遵循 RESTful 风格：

| HTTP 方法 | 含义 | 示例 |
|-----------|------|------|
| `GET` | 读取 | `GET /my-index/_search` |
| `PUT` | 创建或全量更新 | `PUT /my-index` |
| `POST` | 创建或部分更新 | `POST /my-index/_doc` |
| `DELETE` | 删除 | `DELETE /my-index` |

### 请求结构

一个典型的 Easysearch 请求由三部分组成：

```
<HTTP方法> /<索引>/<操作>
{
  <JSON请求体>
}
```

例如，在 `products` 索引中搜索 "手机"：

```
GET /products/_search
{
  "query": {
    "match": {
      "title": "手机"
    }
  }
}
```

### 常用端点

| 端点 | 说明 |
|------|------|
| `/_cat/health` | 集群健康状态（文本格式） |
| `/_cluster/health` | 集群健康状态（JSON） |
| `/_cat/indices` | 索引列表 |
| `/<index>/_search` | 搜索 |
| `/<index>/_doc/<id>` | 读写单条文档 |
| `/_bulk` | 批量操作 |
| `/_analyze` | 分词测试 |

### 响应结构

每个搜索响应都包含以下关键字段：

```json
{
  "took": 5,                    // 耗时（毫秒）
  "timed_out": false,           // 是否超时
  "hits": {
    "total": { "value": 42 },   // 命中总数
    "max_score": 1.5,           // 最高评分
    "hits": [                   // 命中的文档列表
      {
        "_index": "products",
        "_id": "1",
        "_score": 1.5,
        "_source": { ... }      // 原始文档
      }
    ]
  }
}
```

## Query DSL 概览

Query DSL（Domain Specific Language）是 Easysearch 的查询语言，使用 JSON 结构描述查询条件。所有查询都放在 `query` 字段中。

### 两类查询上下文

| 上下文 | 关键字 | 作用 | 是否计算评分 |
|--------|--------|------|:----------:|
| **查询上下文** | `query` | 文档与查询的匹配程度 | 是 |
| **过滤上下文** | `filter` | 文档是否匹配（是/否） | 否（可缓存） |

> 能用 filter 的场景尽量用 filter，不计算评分 + 可缓存 = 更快。

### 基础查询类型

#### match — 全文搜索

对输入文本分词后查询，是最常用的全文查询：

```json
{
  "query": {
    "match": {
      "content": "分布式搜索引擎"
    }
  }
}
```

#### term — 精确匹配

不分词，精确匹配 keyword 字段：

```json
{
  "query": {
    "term": {
      "status": "published"
    }
  }
}
```

#### range — 范围查询

```json
{
  "query": {
    "range": {
      "price": { "gte": 100, "lte": 500 }
    }
  }
}
```

#### bool — 组合查询

通过 `must` / `should` / `must_not` / `filter` 组合多个条件：

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "手机" } }
      ],
      "filter": [
        { "range": { "price": { "lte": 3000 } } },
        { "term": { "brand": "xiaomi" } }
      ]
    }
  }
}
```

### match 与 term 的区别

| | match | term |
|---|-------|------|
| 分词 | 会对输入分词 | 不分词 |
| 适用字段 | `text` | `keyword`、数值、日期 |
| 典型场景 | 全文搜索 | 过滤、精确查找 |

## 分页与排序

```json
{
  "query": { "match_all": {} },
  "from": 0,
  "size": 10,
  "sort": [
    { "created_at": "desc" },
    "_score"
  ]
}
```

- `from` + `size`：适合浅分页（前几百页）
- `search_after`：适合深分页
- `scroll`：适合批量导出

## 高亮

```json
{
  "query": {
    "match": { "content": "搜索引擎" }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

响应中的高亮片段：

```json
"highlight": {
  "content": ["分布式<em>搜索引擎</em>的核心原理"]
}
```

## 聚合（简介）

聚合用于统计分析，可与查询组合：

```json
{
  "size": 0,
  "aggs": {
    "brand_count": {
      "terms": { "field": "brand.keyword" }
    }
  }
}
```

> 详见 [聚合分析]({{< relref "/docs/features/aggregations/" >}})。

## 延伸阅读

- [入门教程]({{< relref "/docs/quick-start/tutorial.md" >}})：完整的动手练习
- [全文搜索]({{< relref "/docs/features/fulltext-search/" >}})：Query DSL 完整指南
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})：过滤与精确查询
- [常用 REST 参数]({{< relref "/docs/api-reference/common-parameters.md" >}})：pretty、human 等通用参数


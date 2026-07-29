---
title: "Query DSL 基础"
date: 0001-01-01
description: "理解查询结构：query/filter、bool 组合与常见写法。"
summary: "Query DSL 基础 #  这一页的目标是：在不记任何 API 细节的前提下，先把 Query DSL 的“骨架”和核心概念讲清楚，后面看具体查询语法就会轻松很多。
JSON 结构而不是&quot;SQL 语句&quot; #  Easysearch 的查询是 JSON 结构，而不是一条字符串语句。一个最小的查询大致长这样：
GET /_search { &#34;query&#34;: { &#34;match&#34;: { &#34;message&#34;: &#34;hello world&#34; } } } 要记住两个层级：
 最外层的 &quot;query&quot;：说明这是一个&quot;查询上下文&quot; &quot;match&quot;、&quot;term&quot;、&quot;range&quot; 等：都是不同类型的&quot;子查询&quot;  查询语句的结构 #  一个查询语句的典型结构：
{ QUERY_NAME: { ARGUMENT: VALUE, ARGUMENT: VALUE,... } } 如果是针对某个字段，那么它的结构如下：
{ QUERY_NAME: { FIELD_NAME: { ARGUMENT: VALUE, ARGUMENT: VALUE,... } } } 例如，使用 match 查询语句来查询 tweet 字段中包含 easysearch 的文档："
---


# Query DSL 基础

这一页的目标是：在不记任何 API 细节的前提下，先把 Query DSL 的“骨架”和核心概念讲清楚，后面看具体查询语法就会轻松很多。

## JSON 结构而不是"SQL 语句"

Easysearch 的查询是 **JSON 结构**，而不是一条字符串语句。一个最小的查询大致长这样：

```json
GET /_search
{
  "query": {
    "match": {
      "message": "hello world"
    }
  }
}
```

要记住两个层级：

- 最外层的 `"query"`：说明这是一个"查询上下文"
- `"match"`、`"term"`、`"range"` 等：都是不同类型的"子查询"

### 查询语句的结构

一个查询语句的典型结构：

```json
{
  QUERY_NAME: {
    ARGUMENT: VALUE,
    ARGUMENT: VALUE,...
  }
}
```

如果是针对某个字段，那么它的结构如下：

```json
{
  QUERY_NAME: {
    FIELD_NAME: {
      ARGUMENT: VALUE,
      ARGUMENT: VALUE,...
    }
  }
}
```

例如，使用 `match` 查询语句来查询 `tweet` 字段中包含 `easysearch` 的文档：

```json
{
  "match": {
    "tweet": "easysearch"
  }
}
```

### 空查询

空查询（`{}`）在功能上等价于使用 `match_all` 查询，正如其名字一样，匹配所有文档：

```json
GET /_search
{
  "query": {
    "match_all": {}
  }
}
```

## Query 上下文 vs Filter 上下文

Query DSL 里有两种“语义上下文”：

- **Query 上下文**：用于计算相关性 `_score`  
  - 典型：`match`、`multi_match` 等全文检索
  - 结果有打分，适合排序
- **Filter 上下文**：只判断“是不是匹配”，不参与 `_score` 计算  
  - 典型：`term`、`range` 作为过滤条件
  - 结果可缓存，通常更快

经验法则：

- “是不是这个用户、是不是这个状态” → filter
- “相关不相关、哪条更像” → query

更直观一点的例子（同样是精确值条件，用在不同上下文）：

```json
{
  "query": {
    "bool": {
      "must": [
        { "term": { "status": "active" } }   // 参与打分
      ],
      "filter": [
        { "term": { "status": "active" } }   // 只过滤，不打分
      ]
    }
  }
}
```

实际场景里，**过滤条件几乎总是放在 `filter` 里**：  
- 逻辑语义更清晰（“必须满足，但不影响相关性”）  
- 容易被缓存，加速高并发请求

## bool 查询：组合一切的基础

大部分复杂查询，最后都会长成一个 `bool`。查询语句可以分为两类：

- **叶子语句（Leaf clauses）**：如 `match` 语句，用于将查询字符串和一个字段（或多个字段）对比
- **复合语句（Compound clauses）**：主要用于合并其它查询语句，如 `bool` 语句

`bool` 查询允许你组合多个查询语句，它接收以下参数：

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "搜索" } }
      ],
      "filter": [
        { "term": { "status": "active" } }
      ],
      "should": [
        { "term": { "tag": "hot" } }
      ],
      "must_not": [
        { "term": { "is_deleted": true } }
      ]
    }
  }
}
```

各个子句的含义：

- `must`：文档必须匹配这些条件才能被包含进来，同时参与评分（query 上下文）
- `filter`：文档必须匹配，但不参与评分（filter 上下文），结果可缓存
- `should`：如果满足这些语句中的任意语句，将增加 `_score`，否则无任何影响。主要用于修正每个文档的相关性得分
- `must_not`：文档必须不匹配这些条件才能被包含进来（排除条件）

> 提示：如果没有 `must` 语句，那么至少需要能够匹配其中的一条 `should` 语句。但如果存在至少一条 `must` 语句，则对 `should` 语句的匹配没有要求。

建议的使用习惯：

- 业务强约束（如状态、租户、时间范围）放到 `filter`
- 影响排序的条件放到 `must` / `should`
- 不希望命中的文档用 `must_not` 排掉

### `minimum_should_match`：控制“should 至少命中几个”

`should` 默认是“加分项”——命中越多 `_score` 越高，但全不命中也不会被过滤掉。  
在有多个 `should` 时，你可以用 `minimum_should_match` 要求“至少命中 N 个”：

```json
{
  "bool": {
    "must": [
      { "match": { "title": "手机" } }
    ],
    "should": [
      { "term": { "brand": "apple" } },
      { "term": { "brand": "xiaomi" } },
      { "term": { "brand": "huawei" } }
    ],
    "minimum_should_match": 1
  }
}
```

这类写法常用于“主条件 + 若干偏好条件”，既保证主条件命中，又能避免“一个偏好都不满足”的结果混进来。

### 复合语句可以嵌套

一条复合语句可以合并任何其它查询语句，包括复合语句。这意味着复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

例如，以下查询是为了找出信件正文包含 `business opportunity` 的星标邮件，或者在收件箱正文包含 `business opportunity` 的非垃圾邮件：

```json
{
  "bool": {
    "must": { "match": { "email": "business opportunity" }},
    "should": [
      { "match": { "starred": true }},
      { "bool": {
        "must": { "match": { "folder": "inbox" }},
        "must_not": { "match": { "spam": true }}
      }}
    ],
    "minimum_should_match": 1
  }
}
```

## 常见 Query 类型的脑图

按用途大致可以这样分类（后续对应到各自章节）：

- **结构化查询**：`term`、`terms`、`range`、`exists` 等
- **全文检索**：`match`、`match_phrase`、`multi_match` 等
- **组合/逻辑查询**：`bool`、`constant_score` 等
- **特殊用途**：`function_score`、`script`、`nested` 等

这一页只关注“壳子”和组合方式，具体语法在后续页面分别展开。

## 写 DSL 时常见几个坑

- **把所有条件都塞进 `must`**  
  - 结果：评分杂乱、缓存利用不好  
  - 建议：能 filter 的尽量用 `filter`

- **过度嵌套 bool，结构难以维护**  
  - 建议：尽量把逻辑整理成“必选/加分/过滤/排除”四类，用一层 bool 表达清楚

- **混用查询类型却不了解语义差异**  
  - 例如对 keyword 字段用全文型查询，或对 text 字段用 term 精确匹配  
  - 建议：先在 Mapping 章节搞清楚字段类型，再决定用哪类查询

下一步可以继续阅读：

- [结构化搜索](./structured-search.md)
- [全文搜索]({{< relref "/docs/features/fulltext-search/_index.md" >}})

## 参考手册（API 与参数）

- [搜索操作概览（功能手册）]({{< relref "docs/features/fulltext-search/_index.md" >}})
- [全文查询（功能手册）]({{< relref "docs/features/fulltext-search/full-text/_index.md" >}})
- [精确/Term 查询（功能手册）]({{< relref "docs/features/query-dsl/term-based-query/_index.md" >}})
- [复合查询（功能手册）]({{< relref "docs/features/query-dsl/compound-query/_index.md" >}})




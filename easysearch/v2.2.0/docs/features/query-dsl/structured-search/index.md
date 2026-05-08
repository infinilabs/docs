---
title: "结构化搜索"
date: 0001-01-01
description: "term/terms/range/exists 等结构化查询与过滤的最佳实践。"
summary: "结构化搜索 #  结构化搜索针对的是“精确可判定”的字段：ID、枚举、时间、数值、状态位等。本页重点讲如何用好 term/terms/range/exists 这几类基础查询，并配合 filter 上下文使用。
适用字段与“不适用”的情况 #  适合用结构化查询的典型字段：
 关键词/编码：订单号、用户 ID、设备 ID、状态码 数值：价格、年龄、计数、评分 时间：创建时间、更新时间、业务时间 标志位/枚举：is_deleted、status、type 等  不适合用结构化查询的字段：
 需要做全文检索的长文本（标题、描述、评论等）
→ 应使用 match/match_phrase 等全文查询，详见“全文检索”章节。  term：精确匹配 #  term 查询不会做分词，也不会自动大小写转换，适合：
 keyword 字段 数值字段 布尔/枚举字段  典型用法（以伪 JSON 示意）：
{ &#34;query&#34;: { &#34;term&#34;: { &#34;status&#34;: &#34;active&#34; } } } 建议：
 对 text 字段做 term 查询通常是错误的（除非你非常明确知道底层分析行为） 业务 ID、编码类字段建议使用 keyword 类型并用 term 查询  terms：在一组值中匹配 #  terms 用于“IN 列表”场景："
---


# 结构化搜索

结构化搜索针对的是“精确可判定”的字段：ID、枚举、时间、数值、状态位等。本页重点讲如何用好 term/terms/range/exists 这几类基础查询，并配合 filter 上下文使用。

## 适用字段与“不适用”的情况

适合用结构化查询的典型字段：

- 关键词/编码：订单号、用户 ID、设备 ID、状态码
- 数值：价格、年龄、计数、评分
- 时间：创建时间、更新时间、业务时间
- 标志位/枚举：is_deleted、status、type 等

不适合用结构化查询的字段：

- 需要做全文检索的长文本（标题、描述、评论等）  
  → 应使用 match/match_phrase 等全文查询，详见“全文检索”章节。

## term：精确匹配

`term` 查询不会做分词，也不会自动大小写转换，适合：

- keyword 字段
- 数值字段
- 布尔/枚举字段

典型用法（以伪 JSON 示意）：

```json
{
  "query": {
    "term": {
      "status": "active"
    }
  }
}
```

建议：

- 对 text 字段做 term 查询通常是错误的（除非你非常明确知道底层分析行为）
- 业务 ID、编码类字段建议使用 keyword 类型并用 term 查询

## terms：在一组值中匹配

`terms` 用于“IN 列表”场景：

```json
{
  "query": {
    "terms": {
      "status": ["active", "pending"]
    }
  }
}
```

典型场景：

- 过滤一组允许状态
- 从一批 ID 中筛选（注意列表长度与性能）

> 经验：非常长的 ID 列表（如几万、几十万）建议通过临时索引/标记字段等方式重构，而不是直接塞进 terms。

## range：范围查询

`range` 用于数值或时间范围：

```json
{
  "query": {
    "range": {
      "created_at": {
        "gte": "2024-01-01T00:00:00Z",
        "lt":  "2024-02-01T00:00:00Z"
      }
    }
  }
}
```

常用比较符号：

- `gt` / `gte`：大于（等于）
- `lt` / `lte`：小于（等于）

建议：

- 时间范围过滤几乎总是应该放在 `filter` 中（不参与打分，可缓存）
- 组合多个 range 时注意边界是否重叠/有缝

## exists：字段存在性判断

`exists` 用于判断字段是否存在：

```json
{
  "query": {
    "exists": {
      "field": "email"
    }
  }
}
```

适合：

- 筛选“有值/缺失”的数据
- 区分“null/缺失” vs “有有效值”

> 小贴士：很多日志/埋点场景里，一部分字段是“有就有，没有就压根不出现在 JSON 里”，这时候 `exists` 比“用特殊值占位再 term 查询”要自然得多。

## 和 bool/filter 结合使用

在实际查询中，这些结构化条件通常放到 `bool.filter` 里：

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "搜索" } }
      ],
      "filter": [
        { "term":  { "status": "active" } },
        { "range": { "created_at": { "gte": "now-7d/d" } } }
      ]
    }
  }
}
```

好处：

- 不参与 `_score` 计算，语义更清晰（它们本来就是“硬过滤条件”）
- 更容易被缓存，提高重复查询性能

### 组合示例：订单列表的典型过滤

一个稍微贴近业务的例子：电商后台的“订单列表”过滤：

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "buyer_keywords": "北京 手机" } }   // 主搜索词：全文检索
      ],
      "filter": [
        { "term":  { "status": "paid" }},                 // 已支付
        { "terms": { "channel": ["app", "web"] }},        // 下单渠道
        {
          "range": { "order_time": {                      // 最近 30 天
            "gte": "now-30d/d"
          }}
        },
        { "term":  { "tenant_id": "t_123" }}              // 多租户隔离
      ]
    }
  }
}
```

可以看到：

- 所有“业务约束/权限/时间范围”都在 `filter` 里
- 只有真正影响排序/相关性的条件放在 `must`

## 典型实践小结

- 所有“业务约束/权限/状态/时间范围”优先用结构化查询表达，并放进 filter
- 尽量避免对 text 字段做 term/terms/range 查询，除非你确实要在分词后的 token 上做精确匹配
- 大批量 ID 过滤要关注 terms 的规模与性能，必要时用索引设计/标记字段重构问题

下一步可以继续阅读：

- [全文搜索]({{< relref "/docs/features/fulltext-search/_index.md" >}})
- [分页与排序]({{< relref "/docs/features/fulltext-search/pagination-and-sorting.md" >}})
- [相关性]({{< relref "/docs/features/fulltext-search/relevance/_index.md" >}})

## 参考手册（API 与参数）

- [搜索操作概览（功能手册）]({{< relref "docs/features/fulltext-search/_index.md" >}})
- [精确/Term 查询（功能手册）]({{< relref "docs/features/query-dsl/term-based-query/_index.md" >}})



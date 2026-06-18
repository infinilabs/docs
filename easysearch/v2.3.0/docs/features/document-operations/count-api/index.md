---
title: "Count API"
date: 0001-01-01
description: "返回匹配查询的文档数量，不返回文档内容。"
summary: "Count API #  返回匹配查询的文档数量，不返回文档内容。
请求格式 #  GET /&lt;index&gt;/_count POST /&lt;index&gt;/_count GET /_count POST /_count 路径参数 #     参数 必需 说明     &lt;index&gt; 否 目标索引，支持逗号分隔多索引和通配符    查询参数 #     参数 类型 默认值 说明     q string — 简单查询字符串   df string — q 参数的默认搜索字段   default_operator string OR q 参数的默认逻辑运算符：AND 或 OR   analyzer string — q 参数使用的分析器   analyze_wildcard boolean false 是否分析通配符   lenient boolean — 宽松解析模式   routing string — 路由值   preference string — 查询偏好   min_score float — 最低分数过滤   terminate_after int 0 每分片最多计数的文档数（0 = 不限制）   expand_wildcards string open 通配符展开方式：open、closed、hidden、all、none   ignore_unavailable boolean false 忽略不存在的索引   allow_no_indices boolean true 允许通配符不匹配任何索引    请求体（可选） #  { &#34;query&#34;: { &#34;term&#34;: { &#34;status&#34;: &#34;published&#34; } } } 示例 #  使用查询字符串："
---


# Count API

返回匹配查询的文档数量，不返回文档内容。

## 请求格式

```
GET /<index>/_count
POST /<index>/_count
GET /_count
POST /_count
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | 否 | 目标索引，支持逗号分隔多索引和通配符 |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `q` | string | — | 简单查询字符串 |
| `df` | string | — | `q` 参数的默认搜索字段 |
| `default_operator` | string | `OR` | `q` 参数的默认逻辑运算符：`AND` 或 `OR` |
| `analyzer` | string | — | `q` 参数使用的分析器 |
| `analyze_wildcard` | boolean | `false` | 是否分析通配符 |
| `lenient` | boolean | — | 宽松解析模式 |
| `routing` | string | — | 路由值 |
| `preference` | string | — | 查询偏好 |
| `min_score` | float | — | 最低分数过滤 |
| `terminate_after` | int | `0` | 每分片最多计数的文档数（0 = 不限制） |
| `expand_wildcards` | string | `open` | 通配符展开方式：`open`、`closed`、`hidden`、`all`、`none` |
| `ignore_unavailable` | boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | boolean | `true` | 允许通配符不匹配任何索引 |

## 请求体（可选）

```json
{
  "query": {
    "term": { "status": "published" }
  }
}
```

## 示例

使用查询字符串：

```json
GET /website/_count?q=title:blog
```

使用请求体查询：

```json
POST /website/_count
{
  "query": {
    "range": {
      "date": { "gte": "2024-01-01" }
    }
  }
}
```

响应：

```json
{
  "count": 42,
  "_shards": {
    "total":      5,
    "successful": 5,
    "skipped":    0,
    "failed":     0
  }
}
```

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 执行完整搜索 | [全文搜索]({{< relref "/docs/features/fulltext-search/_index.md" >}}) |
| 检索单条文档 | [Get API]({{< relref "./get-api.md" >}}) |


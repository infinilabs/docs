---
title: "Get API"
date: 0001-01-01
description: "按 ID 检索文档，支持 _source 过滤、HEAD 存在性检查、仅返回 _source。"
summary: "Get API #  按 ID 检索单条文档，返回完整 _source 及元数据。
请求格式 #  GET /&lt;index&gt;/_doc/&lt;_id&gt; HEAD /&lt;index&gt;/_doc/&lt;_id&gt; 仅返回 _source（不含元数据）：
GET /&lt;index&gt;/_source/&lt;_id&gt; HEAD /&lt;index&gt;/_source/&lt;_id&gt; 路径参数 #     参数 必需 说明     &lt;index&gt; 是 目标索引   &lt;_id&gt; 是 文档 ID    查询参数 #     参数 类型 默认值 说明     _source string/boolean true true/false 启用或禁用 _source 返回，或传逗号分隔字段名仅返回指定字段   _source_includes string — _source 中要包含的字段（逗号分隔，支持通配符）   _source_excludes string — _source 中要排除的字段（逗号分隔，支持通配符）   stored_fields string — 要返回的 stored 字段（逗号分隔）   routing string — 自定义路由值。若文档写入时使用了自定义路由，检索时必须指定相同的值   preference string — 查询偏好。_local = 优先本地分片；_primary = 仅主分片；或自定义字符串   realtime boolean true 实时读取。为 true 时不依赖刷新即可读到最新写入   refresh boolean false 读取前是否强制刷新   version long — 期望的版本号，不匹配时返回 409   version_type string internal 版本类型    示例 #  获取完整文档 #  GET /website/_doc/123 响应："
---


# Get API

按 ID 检索单条文档，返回完整 `_source` 及元数据。

## 请求格式

```
GET /<index>/_doc/<_id>
HEAD /<index>/_doc/<_id>
```

仅返回 `_source`（不含元数据）：

```
GET /<index>/_source/<_id>
HEAD /<index>/_source/<_id>
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | **是** | 目标索引 |
| `<_id>` | **是** | 文档 ID |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `_source` | string/boolean | `true` | `true`/`false` 启用或禁用 `_source` 返回，或传逗号分隔字段名仅返回指定字段 |
| `_source_includes` | string | — | `_source` 中要包含的字段（逗号分隔，支持通配符） |
| `_source_excludes` | string | — | `_source` 中要排除的字段（逗号分隔，支持通配符） |
| `stored_fields` | string | — | 要返回的 stored 字段（逗号分隔） |
| `routing` | string | — | 自定义路由值。若文档写入时使用了自定义路由，检索时**必须**指定相同的值 |
| `preference` | string | — | 查询偏好。`_local` = 优先本地分片；`_primary` = 仅主分片；或自定义字符串 |
| `realtime` | boolean | `true` | 实时读取。为 `true` 时不依赖刷新即可读到最新写入 |
| `refresh` | boolean | `false` | 读取前是否强制刷新 |
| `version` | long | — | 期望的版本号，不匹配时返回 409 |
| `version_type` | string | `internal` | 版本类型 |

## 示例

### 获取完整文档

```json
GET /website/_doc/123
```

响应：

```json
{
  "_index":        "website",
  "_type":         "_doc",
  "_id":           "123",
  "_version":      1,
  "_seq_no":       0,
  "_primary_term": 1,
  "found":         true,
  "_source": {
    "title": "My first blog entry",
    "text":  "Just trying this out...",
    "date":  "2014/01/01"
  }
}
```

文档不存在时 `found` 为 `false`，HTTP 状态码 `404`。

### 只返回部分字段

```json
GET /website/_doc/123?_source_includes=title,date
```

### 只返回 `_source`

```json
GET /website/_source/123
```

直接返回 JSON 文档内容，不含 `_index`、`_version` 等元数据。

### 检查文档是否存在

```json
HEAD /website/_doc/123
```

- 存在 → `200 OK`
- 不存在 → `404 Not Found`

不传输 `_source`，适合只需判断存在性的场景。

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 批量获取多条文档 | [Multi Get API]({{< relref "./multi-get-api.md" >}}) |
| 写入文档 | [Index API]({{< relref "./index-api.md" >}}) |
| `_source` 字段控制 | [_source 与字段存储]({{< relref "/docs/fundamentals/source-and-stored-fields.md" >}}) |


---
title: "Multi Get API"
date: 0001-01-01
description: "一次请求批量获取多条文档，支持跨索引检索和字段过滤。"
summary: "Multi Get API #  一次请求获取多条文档。协调节点按分片拆分请求并行转发，比循环调用 GET 更高效。
请求格式 #  GET /_mget POST /_mget GET /&lt;index&gt;/_mget POST /&lt;index&gt;/_mget 路径参数 #     参数 必需 说明     &lt;index&gt; 否 默认索引。在 URL 中指定后，请求体中的文档可省略 _index    查询参数 #     参数 类型 默认值 说明     routing string — 默认路由值   preference string — 查询偏好   realtime boolean true 实时读取   refresh boolean false 读取前是否刷新   stored_fields string — 默认返回的 stored 字段   _source string/boolean true 默认 _source 过滤   _source_includes string — 默认包含的字段   _source_excludes string — 默认排除的字段    请求体 #  标准格式 #  { &#34;docs&#34;: [ { &#34;_index&#34;: &#34;website&#34;, &#34;_id&#34;: &#34;1&#34; }, { &#34;_index&#34;: &#34;website&#34;, &#34;_id&#34;: &#34;2&#34;, &#34;_source&#34;: [&#34;title&#34;] }, { &#34;_index&#34;: &#34;blog&#34;, &#34;_id&#34;: &#34;3&#34;, &#34;routing&#34;: &#34;user1&#34; } ] } 每条文档可单独指定 _index、_source、routing、stored_fields。"
---


# Multi Get API

一次请求获取多条文档。协调节点按分片拆分请求并行转发，比循环调用 GET 更高效。

## 请求格式

```
GET /_mget
POST /_mget
GET /<index>/_mget
POST /<index>/_mget
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | 否 | 默认索引。在 URL 中指定后，请求体中的文档可省略 `_index` |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `routing` | string | — | 默认路由值 |
| `preference` | string | — | 查询偏好 |
| `realtime` | boolean | `true` | 实时读取 |
| `refresh` | boolean | `false` | 读取前是否刷新 |
| `stored_fields` | string | — | 默认返回的 stored 字段 |
| `_source` | string/boolean | `true` | 默认 `_source` 过滤 |
| `_source_includes` | string | — | 默认包含的字段 |
| `_source_excludes` | string | — | 默认排除的字段 |

## 请求体

### 标准格式

```json
{
  "docs": [
    { "_index": "website", "_id": "1" },
    { "_index": "website", "_id": "2", "_source": ["title"] },
    { "_index": "blog",    "_id": "3", "routing": "user1" }
  ]
}
```

每条文档可单独指定 `_index`、`_source`、`routing`、`stored_fields`。

### 简写格式

当所有文档在同一索引时：

```json
GET /website/_mget
{
  "ids": ["1", "2", "3"]
}
```

## 示例

```json
GET /_mget
{
  "docs": [
    { "_index": "website", "_id": "1" },
    { "_index": "website", "_id": "2", "_source": "title" }
  ]
}
```

响应：

```json
{
  "docs": [
    {
      "_index": "website", "_type": "_doc", "_id": "1",
      "_version": 1, "found": true,
      "_source": { "title": "My first blog entry", "text": "..." }
    },
    {
      "_index": "website", "_type": "_doc", "_id": "2",
      "_version": 1, "found": true,
      "_source": { "title": "My second blog entry" }
    }
  ]
}
```

如果某条文档不存在，对应的 `found` 为 `false`，不影响其他文档的返回。

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 获取单条文档 | [Get API]({{< relref "./get-api.md" >}}) |
| 批量写入操作 | [Bulk API]({{< relref "./bulk-api.md" >}}) |
| 分布式检索过程 | [分布式写入过程]({{< relref "/docs/fundamentals/distributed-write.md" >}}) |


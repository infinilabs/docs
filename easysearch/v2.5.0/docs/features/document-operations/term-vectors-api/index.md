---
title: "Term Vectors API"
date: 0001-01-01
description: "获取文档中特定字段的词项信息（词频、位置、偏移量），用于文本分析与调试。"
summary: "Term Vectors API #  返回文档中特定字段的词项信息（词频、位置、偏移量等），用于文本分析和调试。
请求格式 #  GET /&lt;index&gt;/_termvectors/&lt;_id&gt; POST /&lt;index&gt;/_termvectors/&lt;_id&gt; GET /&lt;index&gt;/_termvectors # 在请求体中提供临时文档 POST /&lt;index&gt;/_termvectors 批量获取：
GET /_mtermvectors POST /_mtermvectors GET /&lt;index&gt;/_mtermvectors POST /&lt;index&gt;/_mtermvectors 路径参数 #     参数 必需 说明     &lt;index&gt; 是 目标索引   &lt;_id&gt; 否 文档 ID。省略时需在请求体中通过 doc 提供临时文档    查询参数 #     参数 类型 默认值 说明     fields string — 逗号分隔的字段列表   offsets boolean true 返回词项偏移量   positions boolean true 返回词项位置   payloads boolean true 返回词项负载   term_statistics boolean false 返回词项的总词频和文档频率   field_statistics boolean true 返回文档计数、文档频率之和、总词频之和   realtime boolean true 实时读取   routing string — 路由值   preference string — 查询偏好    示例 #  GET /website/_termvectors/1?"
---


# Term Vectors API

返回文档中特定字段的词项信息（词频、位置、偏移量等），用于文本分析和调试。

## 请求格式

```
GET /<index>/_termvectors/<_id>
POST /<index>/_termvectors/<_id>
GET /<index>/_termvectors          # 在请求体中提供临时文档
POST /<index>/_termvectors
```

批量获取：

```
GET /_mtermvectors
POST /_mtermvectors
GET /<index>/_mtermvectors
POST /<index>/_mtermvectors
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | **是** | 目标索引 |
| `<_id>` | 否 | 文档 ID。省略时需在请求体中通过 `doc` 提供临时文档 |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `fields` | string | — | 逗号分隔的字段列表 |
| `offsets` | boolean | `true` | 返回词项偏移量 |
| `positions` | boolean | `true` | 返回词项位置 |
| `payloads` | boolean | `true` | 返回词项负载 |
| `term_statistics` | boolean | `false` | 返回词项的总词频和文档频率 |
| `field_statistics` | boolean | `true` | 返回文档计数、文档频率之和、总词频之和 |
| `realtime` | boolean | `true` | 实时读取 |
| `routing` | string | — | 路由值 |
| `preference` | string | — | 查询偏好 |

## 示例

```json
GET /website/_termvectors/1?fields=title
```

在请求体中提供临时文档（无需事先索引）：

```json
POST /website/_termvectors
{
  "doc": {
    "title": "Easysearch is fast"
  },
  "per_field_analyzer": {
    "title": "standard"
  }
}
```

## Multi Term Vectors

```json
POST /_mtermvectors
{
  "docs": [
    { "_index": "website", "_id": "1", "fields": ["title"] },
    { "_index": "website", "_id": "2", "fields": ["title", "text"] }
  ]
}
```

或简写：

```json
POST /website/_mtermvectors
{
  "ids": ["1", "2"],
  "parameters": {
    "fields": ["title"],
    "term_statistics": true
  }
}
```

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 分析器调试 | [映射与分析]({{< relref "/docs/features/mapping-and-analysis/_index.md" >}}) |
| 检索单条文档 | [Get API]({{< relref "./get-api.md" >}}) |


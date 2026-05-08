---
title: "Bulk API"
date: 0001-01-01
description: "在单个请求中执行多个 index、create、update、delete 操作。"
summary: "Bulk API #  在单个请求中执行多个 index、create、update、delete 操作。是高吞吐写入的核心能力。
请求格式 #  POST /_bulk POST /&lt;index&gt;/_bulk 路径参数 #     参数 必需 说明     &lt;index&gt; 否 默认索引。指定后操作行中可省略 _index    查询参数 #     参数 类型 默认值 说明     routing string — 默认路由值   pipeline string — 默认 Ingest Pipeline   refresh string false 所有操作完成后的刷新策略   timeout time 1m 超时时间   wait_for_active_shards string — 活跃分片数量   require_alias boolean — 要求目标必须是别名   _source string/boolean — 默认 _source 过滤   _source_includes string — 默认包含字段   _source_excludes string — 默认排除字段    请求体格式（NDJSON） #  请求体是逐行 JSON（Newline Delimited JSON）格式，每行一个 JSON 对象："
---


# Bulk API

在单个请求中执行多个 `index`、`create`、`update`、`delete` 操作。是高吞吐写入的核心能力。

## 请求格式

```
POST /_bulk
POST /<index>/_bulk
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | 否 | 默认索引。指定后操作行中可省略 `_index` |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `routing` | string | — | 默认路由值 |
| `pipeline` | string | — | 默认 Ingest Pipeline |
| `refresh` | string | `false` | 所有操作完成后的刷新策略 |
| `timeout` | time | `1m` | 超时时间 |
| `wait_for_active_shards` | string | — | 活跃分片数量 |
| `require_alias` | boolean | — | 要求目标必须是别名 |
| `_source` | string/boolean | — | 默认 `_source` 过滤 |
| `_source_includes` | string | — | 默认包含字段 |
| `_source_excludes` | string | — | 默认排除字段 |

## 请求体格式（NDJSON）

请求体是**逐行 JSON**（Newline Delimited JSON）格式，每行一个 JSON 对象：

```
{ "action": { "metadata" } }\n
{ "request_body" }\n
```

**格式要求**：

- 每行必须以换行符 `\n` 结尾，**包括最后一行**
- 行内不能包含未转义的换行符
- 不能使用 pretty 格式
- Content-Type 应为 `application/x-ndjson`

## 操作类型

| 操作 | 说明 | 需要请求体行 |
|------|------|--------------|
| `index` | 写入文档，已存在则覆盖 | **是** |
| `create` | 仅创建，已存在则报 409 | **是** |
| `update` | 部分更新文档 | **是**（`doc` 或 `script`） |
| `delete` | 删除文档 | **否** |

## 操作行元数据

| 字段 | 说明 |
|------|------|
| `_index` | 目标索引（URL 已指定时可省略） |
| `_id` | 文档 ID（`index` 和 `create` 可省略以自动生成） |
| `routing` | 路由值 |
| `pipeline` | Ingest Pipeline |
| `require_alias` | 要求目标是别名 |
| `_retry_on_conflict` | `update` 操作的冲突重试次数 |

## 示例

```json
POST /_bulk
{ "delete": { "_index": "website", "_id": "123" } }
{ "create": { "_index": "website", "_id": "123" } }
{ "title": "My first blog post" }
{ "index":  { "_index": "website" } }
{ "title": "My second blog post" }
{ "update": { "_index": "website", "_id": "123", "_retry_on_conflict": 3 } }
{ "doc": { "title": "My updated blog post" } }
```

### 指定默认索引

```json
POST /website/_bulk
{ "index": {} }
{ "title": "Event logged in" }
{ "index": {} }
{ "title": "Another event" }
```

## 响应

```json
{
  "took":   4,
  "errors": false,
  "items": [
    { "delete": { "_index": "website", "_id": "123", "_version": 2, "status": 200, "result": "deleted" } },
    { "create": { "_index": "website", "_id": "123", "_version": 3, "status": 201, "result": "created" } },
    { "create": { "_index": "website", "_id": "EiwfApScQiiy7TIKFxRCTw", "_version": 1, "status": 201, "result": "created" } },
    { "update": { "_index": "website", "_id": "123", "_version": 4, "status": 200, "result": "updated" } }
  ]
}
```

- `errors: false`：所有子操作均成功
- `errors: true`：至少有一条失败，需逐条检查 `items` 中的 `status` 与 `error`

### 部分失败示例

```json
{
  "took": 3,
  "errors": true,
  "items": [
    {
      "create": {
        "_index": "website", "_id": "123",
        "status": 409,
        "error": { "type": "version_conflict_engine_exception", "reason": "..." }
      }
    },
    {
      "index": {
        "_index": "website", "_id": "124",
        "_version": 1, "status": 201, "result": "created"
      }
    }
  ]
}
```

> **Bulk 不是事务**。每条子操作独立执行、独立成功或失败。调用方需逐条检查结果并决定是否重试。

## 批次大小建议

没有固定最佳值，取决于硬件配置、文档大小、集群负载。推荐：

- 从 **1,000–5,000 条**文档 / 每批 **5–15 MB** 开始
- 逐步增大，观察节点 CPU/内存/IO/延迟
- 当吞吐量不再上升或开始下降时，即为当前环境的上限

Bulk 请求在协调节点一次性驻留内存，过大的批次会占用过多内存并影响 GC。

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 单条写入 | [Index API]({{< relref "./index-api.md" >}}) |
| 批量检索 | [Multi Get API]({{< relref "./multi-get-api.md" >}}) |
| 写入链路预处理 | [摄取管道]({{< relref "/docs/features/ingest-pipelines/_index.md" >}}) |
| 分布式写入过程 | [分布式写入过程]({{< relref "/docs/fundamentals/distributed-write.md" >}}) |


---
title: "Index API"
date: 0001-01-01
description: "将 JSON 文档写入索引，支持指定 ID、自动生成 ID、仅创建模式。"
summary: "Index API #  将一个 JSON 文档写入指定索引。若文档 ID 已存在，则整文档覆盖并递增 _version。
请求格式 #  PUT /&lt;index&gt;/_doc/&lt;_id&gt; POST /&lt;index&gt;/_doc/&lt;_id&gt; POST /&lt;index&gt;/_doc # 自动生成 _id 仅创建（文档存在时返回 409 Conflict）：
PUT /&lt;index&gt;/_create/&lt;_id&gt; POST /&lt;index&gt;/_create/&lt;_id&gt; 路径参数 #     参数 必需 说明     &lt;index&gt; 是 目标索引名称   &lt;_id&gt; 否 文档 ID。使用 POST /&lt;index&gt;/_doc 时可省略，由系统自动生成    查询参数 #     参数 类型 默认值 说明     op_type string index 写入类型。index = 创建或覆盖；create = 仅创建（已存在时返回 409）。使用 _create 端点时强制为 create   routing string — 自定义路由值，决定文档落入哪个分片   pipeline string — 写入前执行的 Ingest Pipeline 名称   refresh string false 写入后是否刷新。true = 立即刷新；wait_for = 等待下一次自动刷新完成后返回；false = 不等待   timeout time 1m 等待主分片可用的超时时间   version long — 用于外部版本控制的版本号   version_type string internal 版本类型：internal、external、external_gte   if_seq_no long — 乐观并发控制：仅当文档的 _seq_no 等于此值时才执行写入   if_primary_term long — 乐观并发控制：仅当文档的 _primary_term 等于此值时才执行写入   wait_for_active_shards string 1 写入前需要的活跃分片数量。1 = 仅主分片；all = 全部分片   require_alias boolean false 为 true 时要求 &lt;index&gt; 必须是别名，否则返回错误    请求体 #  完整的 JSON 文档，即 _source 的内容。"
---


# Index API

将一个 JSON 文档写入指定索引。若文档 ID 已存在，则整文档覆盖并递增 `_version`。

## 请求格式

```
PUT /<index>/_doc/<_id>
POST /<index>/_doc/<_id>
POST /<index>/_doc            # 自动生成 _id
```

仅创建（文档存在时返回 `409 Conflict`）：

```
PUT /<index>/_create/<_id>
POST /<index>/_create/<_id>
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | **是** | 目标索引名称 |
| `<_id>` | 否 | 文档 ID。使用 `POST /<index>/_doc` 时可省略，由系统自动生成 |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `op_type` | string | `index` | 写入类型。`index` = 创建或覆盖；`create` = 仅创建（已存在时返回 409）。使用 `_create` 端点时强制为 `create` |
| `routing` | string | — | 自定义路由值，决定文档落入哪个分片 |
| `pipeline` | string | — | 写入前执行的 [Ingest Pipeline]({{< relref "/docs/features/ingest-pipelines/_index.md" >}}) 名称 |
| `refresh` | string | `false` | 写入后是否刷新。`true` = 立即刷新；`wait_for` = 等待下一次自动刷新完成后返回；`false` = 不等待 |
| `timeout` | time | `1m` | 等待主分片可用的超时时间 |
| `version` | long | — | 用于外部版本控制的版本号 |
| `version_type` | string | `internal` | 版本类型：`internal`、`external`、`external_gte` |
| `if_seq_no` | long | — | 乐观并发控制：仅当文档的 `_seq_no` 等于此值时才执行写入 |
| `if_primary_term` | long | — | 乐观并发控制：仅当文档的 `_primary_term` 等于此值时才执行写入 |
| `wait_for_active_shards` | string | `1` | 写入前需要的活跃分片数量。`1` = 仅主分片；`all` = 全部分片 |
| `require_alias` | boolean | `false` | 为 `true` 时要求 `<index>` 必须是别名，否则返回错误 |

## 请求体

完整的 JSON 文档，即 `_source` 的内容。

## 示例

### 指定 ID 写入

```json
PUT /website/_doc/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}
```

响应：

```json
{
  "_index":   "website",
  "_type":    "_doc",
  "_id":      "123",
  "_version": 1,
  "result":   "created",
  "_shards": {
    "total":      2,
    "successful": 1,
    "failed":     0
  },
  "_seq_no":       0,
  "_primary_term": 1
}
```

再次 PUT 同一 `_id` 会覆盖旧文档：`result` 变为 `"updated"`，`_version` 递增。

> **实现细节**：更新不是原地改写，而是将旧文档标记删除，再写入一条新文档。后台通过段合并（segment merge）彻底清理旧版本。

### 自动生成 ID

```json
POST /website/_doc
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

响应中 `_id` 由系统生成（URL 安全、全局唯一）：

```json
{
  "_index":   "website",
  "_type":    "_doc",
  "_id":      "AVFgSgVHUP18jI2wRx0w",
  "_version": 1,
  "result":   "created"
}
```

> 自动 ID 无法保证幂等写入。如果请求超时后重试，可能会产生两条不同 `_id` 的重复文档。需要幂等语义时请自行指定 `_id`。

### 仅创建（禁止覆盖）

```json
PUT /website/_doc/123?op_type=create
{ "title": "My first blog entry" }
```

或等价写法：

```json
PUT /website/_doc/123/_create
{ "title": "My first blog entry" }
```

- 文档不存在 → `201 Created`
- 文档已存在 → `409 Conflict`，原文档不受影响

适用场景：日志/事件流、财务流水等"只允许插入不允许覆盖"的业务。

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 按 ID 检索文档 | [Get API]({{< relref "./get-api.md" >}}) |
| 部分更新文档 | [Update API]({{< relref "./update-api.md" >}}) |
| 批量写入 | [Bulk API]({{< relref "./bulk-api.md" >}}) |
| 写入的分布式协调 | [分布式写入过程]({{< relref "/docs/fundamentals/distributed-write.md" >}}) |
| 乐观并发控制 | [并发控制与版本]({{< relref "/docs/fundamentals/concurrency-and-versioning.md" >}}) |


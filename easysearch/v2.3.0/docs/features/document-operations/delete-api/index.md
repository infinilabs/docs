---
title: "Delete API"
date: 0001-01-01
description: "按 ID 删除单条文档。"
summary: "Delete API #  按 ID 删除单条文档。
请求格式 #  DELETE /&lt;index&gt;/_doc/&lt;_id&gt; 路径参数 #     参数 必需 说明     &lt;index&gt; 是 目标索引   &lt;_id&gt; 是 文档 ID    查询参数 #     参数 类型 默认值 说明     routing string — 自定义路由值   timeout time 1m 等待主分片可用的超时   refresh string false 删除后刷新策略   version long — 期望版本号   version_type string internal 版本类型   if_seq_no long — 乐观并发控制   if_primary_term long — 乐观并发控制   wait_for_active_shards string — 活跃分片数量    示例 #  DELETE /website/_doc/123 响应："
---


# Delete API

按 ID 删除单条文档。

## 请求格式

```
DELETE /<index>/_doc/<_id>
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | **是** | 目标索引 |
| `<_id>` | **是** | 文档 ID |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `routing` | string | — | 自定义路由值 |
| `timeout` | time | `1m` | 等待主分片可用的超时 |
| `refresh` | string | `false` | 删除后刷新策略 |
| `version` | long | — | 期望版本号 |
| `version_type` | string | `internal` | 版本类型 |
| `if_seq_no` | long | — | 乐观并发控制 |
| `if_primary_term` | long | — | 乐观并发控制 |
| `wait_for_active_shards` | string | — | 活跃分片数量 |

## 示例

```json
DELETE /website/_doc/123
```

响应：

```json
{
  "_index":   "website",
  "_type":    "_doc",
  "_id":      "123",
  "_version": 2,
  "result":   "deleted"
}
```

> 删除后 `_version` 递增。即使文档被删除后重新写入同一 `_id`，`_version` 仍会继续递增。

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 按查询批量删除 | [Delete by Query]({{< relref "./delete-by-query.md" >}}) |
| 写入文档 | [Index API]({{< relref "./index-api.md" >}}) |
| 乐观并发控制 | [并发控制与版本]({{< relref "/docs/fundamentals/concurrency-and-versioning.md" >}}) |


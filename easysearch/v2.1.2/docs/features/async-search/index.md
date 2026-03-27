---
title: "异步搜索"
date: 0001-01-01
description: "针对大查询和深度聚合，提供非阻塞的后台检索体验，避免长时间占用前端连接。"
summary: "异步搜索 #  当需要对海量数据执行复杂聚合或跨大时间范围查询时，同步搜索可能因超时而失败。Easysearch 异步搜索（Async Search）允许将这类&quot;大查询&quot;提交到后台执行，客户端无需保持连接等待，可以随时轮询检查进度并获取部分或最终结果。
 适用场景 #     场景 说明     深度聚合分析 对数亿条日志做多维分组聚合，耗时可能达分钟级   大跨度时间查询 查询数月甚至数年的历史数据   快照搜索 在对象存储上搜索冷数据，I/O 延迟较高   跨集群搜索 同时检索多个远程集群，网络延迟不可控   报表与导出 后台生成大规模报表，前端异步展示进度     基本用法 #  提交异步搜索 #  使用 _async_search 端点提交查询：
POST /my-index/_async_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;status_over_time&#34;: { &#34;date_histogram&#34;: { &#34;field&#34;: &#34;@timestamp&#34;, &#34;fixed_interval&#34;: &#34;1h&#34; }, &#34;aggs&#34;: { &#34;avg_response&#34;: { &#34;avg&#34;: { &#34;field&#34;: &#34;response_time&#34; } } } } } } 返回结果中包含一个异步搜索 ID："
---


# 异步搜索

当需要对海量数据执行复杂聚合或跨大时间范围查询时，同步搜索可能因超时而失败。Easysearch 异步搜索（Async Search）允许将这类"大查询"提交到后台执行，客户端无需保持连接等待，可以随时轮询检查进度并获取部分或最终结果。

---

## 适用场景

| 场景 | 说明 |
|------|------|
| **深度聚合分析** | 对数亿条日志做多维分组聚合，耗时可能达分钟级 |
| **大跨度时间查询** | 查询数月甚至数年的历史数据 |
| **快照搜索** | 在对象存储上搜索冷数据，I/O 延迟较高 |
| **跨集群搜索** | 同时检索多个远程集群，网络延迟不可控 |
| **报表与导出** | 后台生成大规模报表，前端异步展示进度 |

---

## 基本用法

### 提交异步搜索

使用 `_async_search` 端点提交查询：

```json
POST /my-index/_async_search
{
  "size": 0,
  "aggs": {
    "status_over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "1h"
      },
      "aggs": {
        "avg_response": {
          "avg": { "field": "response_time" }
        }
      }
    }
  }
}
```

返回结果中包含一个异步搜索 ID：

```json
{
  "id": "FmRGZGFhbVF...",
  "is_partial": true,
  "is_running": true,
  "response": { ... }
}
```

### 查询进度

使用异步搜索 ID 轮询进度：

```json
GET /_async_search/FmRGZGFhbVF...
```

当 `is_running` 为 `false` 时，搜索完成。

### 获取最终结果

搜索完成后，返回完整结果：

```json
{
  "id": "FmRGZGFhbVF...",
  "is_partial": false,
  "is_running": false,
  "response": {
    "took": 45230,
    "hits": { ... },
    "aggregations": { ... }
  }
}
```

### 删除异步搜索

不再需要的异步搜索任务可以手动删除以释放资源：

```json
DELETE /_async_search/FmRGZGFhbVF...
```

---

## 关键参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `wait_for_completion_timeout` | 初次提交时等待同步返回的时间，超时后转为异步 | `1s` |
| `keep_on_completion` | 查询完成后是否保留结果 | `false` |
| `keep_alive` | 异步搜索结果的保留时间 | `5d` |

示例——提交时最多等待 5 秒，完成后保留结果 1 天：

```json
POST /my-index/_async_search?wait_for_completion_timeout=5s&keep_on_completion=true&keep_alive=1d
{
  "query": { "match_all": {} }
}
```

---

## 与快照搜索结合

异步搜索与 [快照搜索]({{< relref "./data-retention/searchable-snapshot" >}}) 天然互补：

- 快照搜索面向大时间跨度和冷数据，I/O 延迟较高
- 通过异步搜索执行，避免长时间查询阻塞系统
- 特别适合审计型、合规型的历史数据分析任务

```json
POST /my-snapshot-index/_async_search
{
  "size": 0,
  "query": {
    "range": {
      "@timestamp": {
        "gte": "2024-01-01",
        "lte": "2024-12-31"
      }
    }
  },
  "aggs": {
    "monthly_stats": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      }
    }
  }
}
```

---

## 相关文档

- [快照搜索]({{< relref "./data-retention/searchable-snapshot" >}})
- [跨集群搜索]({{< relref "./cross-cluster-search" >}})

---

## API 参考

> 以下为异步搜索 REST API 的完整参数与响应字段说明。引入版本 1.11.0。

### 提交异步搜索

```
POST {index}/_async_search
```

**请求参数**

| 参数 | 说明 | 默认值 | 必填 |
|------|------|--------|------|
| `wait_for_completion_timeout` | 等待同步返回的时间，超时后转为异步。最大 300 秒 | `1s` | 否 |
| `keep_on_completion` | 搜索完成后是否保留结果 | `false` | 否 |
| `keep_alive` | 结果保留时间（含查询执行时间），超时后自动删除 | `12h` | 否 |
| `index` | 搜索的索引名称，支持逗号分隔或通配符 | 所有索引 | 否 |

**请求示例**

```json
POST test-index/_async_search?wait_for_completion_timeout=1ms&keep_on_completion=true
{
  "query": {
    "match": {
      "name": "张三"
    }
  }
}
```

**响应示例**

```json
{
  "id": "FmFqN0llTXlKVHF5...",
  "state": "RUNNING",
  "start_time_in_millis": 1740714470020,
  "expiration_time_in_millis": 1740800870020,
  "response": {
    "took": 0,
    "timed_out": false,
    "_shards": { "total": 1, "successful": 0, "skipped": 0, "failed": 0 },
    "hits": { "max_score": null, "hits": [] }
  }
}
```

**响应参数**

| 参数 | 说明 |
|------|------|
| `id` | 异步搜索 ID，用于轮询进度、获取结果或删除。如果在超时内完成则不返回 |
| `state` | 搜索状态：`RUNNING`、`SUCCEEDED`、`FAILED`、`PERSISTING`、`PERSIST_SUCCEEDED`、`PERSIST_FAILED`、`CLOSED`、`STORE_RESIDENT` |
| `start_time_in_millis` | 开始时间（毫秒） |
| `expiration_time_in_millis` | 过期时间（毫秒） |
| `took` | 搜索总耗时 |
| `response` | 搜索响应体 |
| `num_reduce_phases` | 协调节点聚合分片结果的次数（默认 5），数值增加表示有新结果 |

### 获取部分结果

```
GET _async_search/<ID>
```

`state` 变为 `STORE_RESIDENT` 表示结果已成功持久化。可使用 `wait_for_completion_timeout` 参数轮询等待。

### 删除异步搜索

```
DELETE _async_search/<ID>
```

- 搜索进行中：取消搜索
- 搜索已完成：删除保存的结果

### 监控统计信息

```
GET _async_search/stats
```

**统计字段**

| 参数 | 说明 |
|------|------|
| `submitted` | 已提交的请求数 |
| `initialized` | 已初始化的请求数 |
| `running_current` | 当前运行中的请求数 |
| `search_completed` | 成功完成的请求数 |
| `search_failed` | 失败的请求数 |
| `persisted` | 成功持久化的请求数 |
| `persist_failed` | 持久化失败的请求数 |
| `rejected` | 被拒绝的请求数 |
| `cancelled` | 被取消的请求数 |

### 集群设置

异步搜索相关设置为动态设置，无需重启集群：

```json
PUT _cluster/settings
{
  "transient": {
    "async_search.max_wait_for_completion_timeout": "5m"
  }
}
```

| 设置 | 默认值 | 说明 |
|------|--------|------|
| `async_search.max_search_running_time` | `12h` | 搜索最长运行时间，超时自动终止 |
| `async_search.node_concurrent_running_searches` | `20` | 每节点同时运行的异步搜索数 |
| `async_search.max_keep_alive` | `5d` | 结果最长保留时间 |
| `async_search.max_wait_for_completion_timeout` | `1m` | `wait_for_completion_timeout` 最大值 |
| `async_search.persist_search_failures` | `false` | 是否持久化搜索失败的结果 |

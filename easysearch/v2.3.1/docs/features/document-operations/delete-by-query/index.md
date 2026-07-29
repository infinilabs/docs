---
title: "Delete by Query"
date: 0001-01-01
description: "按查询条件批量删除文档，支持异步执行、流量控制与并行切片。"
summary: "Delete by Query #  Delete by Query 删除所有匹配查询条件的文档。 适用于批量清理过期数据、删除特定条件的文档等场景。
内部流程：scroll 遍历匹配文档 → 逐批执行 bulk delete → 返回统计结果。
 请求格式 #  POST /&lt;index&gt;/_delete_by_query 路径参数 #     参数 必需 说明     &lt;index&gt; 是 目标索引，支持逗号分隔多索引和通配符    查询参数 #     参数 类型 默认值 说明     refresh boolean false 操作完成后是否刷新受影响的分片   timeout time 1m 超时时间   wait_for_completion boolean true 为 true 时同步等待；为 false 时立即返回 task ID   wait_for_active_shards string — 操作前需要的活跃分片数量   requests_per_second float 无限制 每秒请求数限制。-1 = 不限制   slices int/string 1 并行切片数。&quot;auto&quot; = 按分片数自动拆分   scroll time — scroll 上下文存活时间   scroll_size int 1000 每批 scroll 获取的文档数   conflicts string abort 版本冲突处理：abort = 中止；proceed = 跳过继续   max_docs int 全部 最多删除的文档数量   search_timeout time — 搜索阶段超时   q string — 简单查询字符串   df string — q 参数的默认字段   default_operator string OR q 参数的默认运算符   analyzer string — q 参数使用的分析器   analyze_wildcard boolean false 是否分析通配符   lenient boolean — 宽松解析模式   routing string — 路由值   preference string — 查询偏好   terminate_after int 0 每分片最多处理的文档数   expand_wildcards string open 通配符展开方式   ignore_unavailable boolean false 忽略不存在的索引   allow_no_indices boolean true 允许通配符不匹配任何索引    请求体 #  { &#34;query&#34;: { ."
---


# Delete by Query

Delete by Query 删除所有匹配查询条件的文档。
适用于批量清理过期数据、删除特定条件的文档等场景。

内部流程：scroll 遍历匹配文档 → 逐批执行 bulk delete → 返回统计结果。

---

## 请求格式

```
POST /<index>/_delete_by_query
```

### 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | **是** | 目标索引，支持逗号分隔多索引和通配符 |

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `refresh` | boolean | `false` | 操作完成后是否刷新受影响的分片 |
| `timeout` | time | `1m` | 超时时间 |
| `wait_for_completion` | boolean | `true` | 为 `true` 时同步等待；为 `false` 时立即返回 task ID |
| `wait_for_active_shards` | string | — | 操作前需要的活跃分片数量 |
| `requests_per_second` | float | 无限制 | 每秒请求数限制。`-1` = 不限制 |
| `slices` | int/string | `1` | 并行切片数。`"auto"` = 按分片数自动拆分 |
| `scroll` | time | — | scroll 上下文存活时间 |
| `scroll_size` | int | `1000` | 每批 scroll 获取的文档数 |
| `conflicts` | string | `abort` | 版本冲突处理：`abort` = 中止；`proceed` = 跳过继续 |
| `max_docs` | int | 全部 | 最多删除的文档数量 |
| `search_timeout` | time | — | 搜索阶段超时 |
| `q` | string | — | 简单查询字符串 |
| `df` | string | — | `q` 参数的默认字段 |
| `default_operator` | string | `OR` | `q` 参数的默认运算符 |
| `analyzer` | string | — | `q` 参数使用的分析器 |
| `analyze_wildcard` | boolean | `false` | 是否分析通配符 |
| `lenient` | boolean | — | 宽松解析模式 |
| `routing` | string | — | 路由值 |
| `preference` | string | — | 查询偏好 |
| `terminate_after` | int | `0` | 每分片最多处理的文档数 |
| `expand_wildcards` | string | `open` | 通配符展开方式 |
| `ignore_unavailable` | boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | boolean | `true` | 允许通配符不匹配任何索引 |

### 请求体

```json
{
  "query": { ... },
  "max_docs": 1000,
  "conflicts": "proceed"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `query` | object | 筛选条件，语法同 [Query DSL]({{< relref "/docs/features/query-dsl/_index.md" >}})。**必需** |
| `max_docs` | int | 最多删除的文档数 |
| `conflicts` | string | 冲突处理策略 |

---

## 示例

### 基本用法

删除所有 `status` 为 `"expired"` 的文档：

```json
POST /website/_delete_by_query
{
  "query": {
    "term": { "status": "expired" }
  }
}
```

### 删除索引中的所有文档

```json
POST /website/_delete_by_query
{
  "query": { "match_all": {} }
}
```

> 如果需要清空整个索引，删除并重建索引通常比 delete_by_query 更高效。

### 使用查询字符串

```json
POST /website/_delete_by_query?q=status:expired
```

### 跳过版本冲突

并发场景下，其他进程可能同时修改了文档。设置 `conflicts=proceed` 跳过冲突继续删除：

```json
POST /website/_delete_by_query?conflicts=proceed
{
  "query": {
    "range": {
      "date": { "lt": "2023-01-01" }
    }
  }
}
```

### 流量控制

限制每秒处理 200 条，降低集群负载：

```json
POST /website/_delete_by_query?requests_per_second=200
{
  "query": {
    "term": { "status": "archived" }
  }
}
```

### 异步执行

```json
POST /website/_delete_by_query?wait_for_completion=false
{
  "query": {
    "range": { "date": { "lt": "2022-01-01" } }
  }
}
```

响应返回 task ID：

```json
{
  "task": "node_id:task_id"
}
```

查询进度：

```json
GET /_tasks/node_id:task_id
```

### 并行切片

```json
POST /website/_delete_by_query?slices=auto
{
  "query": {
    "term": { "status": "expired" }
  }
}
```

### 限制删除数量

最多删除 1000 条匹配文档：

```json
POST /website/_delete_by_query
{
  "query": { "match_all": {} },
  "max_docs": 1000
}
```

---

## 响应字段

```json
{
  "took":                 83,
  "timed_out":            false,
  "total":                100,
  "deleted":              100,
  "batches":              1,
  "version_conflicts":    0,
  "noops":                0,
  "retries": {
    "bulk":   0,
    "search": 0
  },
  "failures": []
}
```

| 字段 | 说明 |
|------|------|
| `took` | 操作耗时（毫秒） |
| `total` | 匹配的文档总数 |
| `deleted` | 成功删除的文档数 |
| `version_conflicts` | 版本冲突次数 |
| `noops` | 无操作次数 |
| `batches` | scroll 批次数 |
| `retries.bulk` | bulk 重试次数 |
| `retries.search` | search 重试次数 |
| `failures` | 失败详情数组 |

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 单条文档删除 | [Delete API]({{< relref "./delete-api.md" >}}) |
| 按查询条件批量更新 | [Update by Query]({{< relref "./update-by-query.md" >}}) |
| 跨索引迁移数据 | [Reindex]({{< relref "/docs/operations/data-management/reindex.md" >}}) |
| 数据生命周期管理 | [数据生命周期与保留策略]({{< relref "/docs/best-practices/data-lifecycle.md" >}}) |


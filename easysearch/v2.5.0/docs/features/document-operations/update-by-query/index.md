---
title: "Update by Query"
date: 0001-01-01
description: "按查询条件批量更新文档，支持脚本修改、异步执行与流量控制。"
summary: "Update by Query #  Update by Query 对所有匹配查询条件的文档执行更新操作。 常用于批量修改字段值、数据迁移补丁、通过脚本做批量计算等场景。
内部流程：scroll 遍历匹配文档 → 逐批执行 bulk update → 返回统计结果。
 请求格式 #  POST /&lt;index&gt;/_update_by_query 路径参数 #     参数 必需 说明     &lt;index&gt; 是 目标索引，支持逗号分隔多索引和通配符    查询参数 #     参数 类型 默认值 说明     refresh boolean false 操作完成后是否刷新受影响的分片   timeout time 1m 超时时间   wait_for_completion boolean true 为 true 时同步等待完成后返回结果；为 false 时立即返回 task ID，可通过 Task API 查询进度   wait_for_active_shards string — 操作前需要的活跃分片数量   requests_per_second float 无限制 每秒请求数限制（流量控制）。-1 = 不限制   slices int/string 1 并行切片数。&quot;auto&quot; = 自动按分片数拆分   scroll time — scroll 上下文存活时间（如 5m）   scroll_size int 1000 每批 scroll 获取的文档数   conflicts string abort 版本冲突处理方式：abort = 遇到冲突即中止；proceed = 跳过冲突继续执行   max_docs int 全部 最多处理的文档数量   pipeline string — 对匹配文档执行的 Ingest Pipeline   search_timeout time — 搜索阶段超时   q string — 简单查询字符串（替代请求体中的 query）   df string — q 参数的默认字段   default_operator string OR q 参数的默认运算符   analyzer string — q 参数使用的分析器   analyze_wildcard boolean false 是否分析通配符   lenient boolean — 宽松解析模式   routing string — 路由值   preference string — 查询偏好   terminate_after int 0 每分片最多处理的文档数（0 = 不限制）   expand_wildcards string open 通配符展开：open、closed、hidden、all、none   ignore_unavailable boolean false 忽略不存在的索引   allow_no_indices boolean true 允许通配符不匹配任何索引    请求体 #  { &#34;query&#34;: { ."
---


# Update by Query

Update by Query 对所有匹配查询条件的文档执行更新操作。
常用于批量修改字段值、数据迁移补丁、通过脚本做批量计算等场景。

内部流程：scroll 遍历匹配文档 → 逐批执行 bulk update → 返回统计结果。

---

## 请求格式

```
POST /<index>/_update_by_query
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
| `wait_for_completion` | boolean | `true` | 为 `true` 时同步等待完成后返回结果；为 `false` 时立即返回 task ID，可通过 Task API 查询进度 |
| `wait_for_active_shards` | string | — | 操作前需要的活跃分片数量 |
| `requests_per_second` | float | 无限制 | 每秒请求数限制（流量控制）。`-1` = 不限制 |
| `slices` | int/string | `1` | 并行切片数。`"auto"` = 自动按分片数拆分 |
| `scroll` | time | — | scroll 上下文存活时间（如 `5m`） |
| `scroll_size` | int | `1000` | 每批 scroll 获取的文档数 |
| `conflicts` | string | `abort` | 版本冲突处理方式：`abort` = 遇到冲突即中止；`proceed` = 跳过冲突继续执行 |
| `max_docs` | int | 全部 | 最多处理的文档数量 |
| `pipeline` | string | — | 对匹配文档执行的 Ingest Pipeline |
| `search_timeout` | time | — | 搜索阶段超时 |
| `q` | string | — | 简单查询字符串（替代请求体中的 `query`） |
| `df` | string | — | `q` 参数的默认字段 |
| `default_operator` | string | `OR` | `q` 参数的默认运算符 |
| `analyzer` | string | — | `q` 参数使用的分析器 |
| `analyze_wildcard` | boolean | `false` | 是否分析通配符 |
| `lenient` | boolean | — | 宽松解析模式 |
| `routing` | string | — | 路由值 |
| `preference` | string | — | 查询偏好 |
| `terminate_after` | int | `0` | 每分片最多处理的文档数（0 = 不限制） |
| `expand_wildcards` | string | `open` | 通配符展开：`open`、`closed`、`hidden`、`all`、`none` |
| `ignore_unavailable` | boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | boolean | `true` | 允许通配符不匹配任何索引 |

### 请求体

```json
{
  "query": { ... },
  "script": {
    "source": "...",
    "lang": "painless",
    "params": { ... }
  },
  "max_docs": 1000,
  "conflicts": "proceed"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `query` | object | 筛选条件，语法同 [Query DSL]({{< relref "/docs/features/query-dsl/_index.md" >}})。省略时匹配所有文档 |
| `script` | object | 更新脚本。通过 `ctx._source` 访问和修改文档字段 |
| `max_docs` | int | 最多处理的文档数 |
| `conflicts` | string | 冲突处理策略 |

> 不提供 `script` 时，Update by Query 仅对匹配文档触发重建索引（re-index in place），常用于 mapping 变更后重新分析已有文档。

---

## 示例

### 基本用法：给所有匹配文档添加字段

```json
POST /website/_update_by_query
{
  "query": {
    "term": { "status": "draft" }
  },
  "script": {
    "source": "ctx._source.status = 'published'",
    "lang":   "painless"
  }
}
```

### 不带脚本：重建索引

mapping 变更后，对所有文档触发重新索引：

```json
POST /website/_update_by_query
{
  "query": { "match_all": {} }
}
```

### 流量控制

限制每秒处理 500 条，避免影响线上查询：

```json
POST /website/_update_by_query?requests_per_second=500&conflicts=proceed
{
  "query": { "match_all": {} },
  "script": {
    "source": "ctx._source.migrated = true",
    "lang":   "painless"
  }
}
```

### 异步执行

大批量更新时，使用异步模式：

```json
POST /website/_update_by_query?wait_for_completion=false
{
  "query": { "match_all": {} },
  "script": {
    "source": "ctx._source.version = 2",
    "lang":   "painless"
  }
}
```

响应返回 task ID：

```json
{
  "task": "node_id:task_id"
}
```

通过 Task API 查询进度：

```json
GET /_tasks/node_id:task_id
```

### 并行切片

使用 `slices=auto` 按分片数自动并行处理：

```json
POST /website/_update_by_query?slices=auto
{
  "query": { "match_all": {} },
  "script": {
    "source": "ctx._source.processed = true",
    "lang":   "painless"
  }
}
```

### 通过 Pipeline 更新

利用已有 Ingest Pipeline 处理匹配文档：

```json
POST /website/_update_by_query?pipeline=my_pipeline
{
  "query": { "match_all": {} }
}
```

---

## 响应字段

```json
{
  "took":                 147,
  "timed_out":            false,
  "total":                5,
  "updated":              5,
  "deleted":              0,
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
| `updated` | 成功更新的文档数 |
| `deleted` | 被脚本删除的文档数（脚本设置 `ctx.op = "delete"` 时） |
| `version_conflicts` | 版本冲突次数 |
| `noops` | 无操作次数（脚本设置 `ctx.op = "noop"` 时） |
| `batches` | scroll 批次数 |
| `retries.bulk` | bulk 重试次数 |
| `retries.search` | search 重试次数 |
| `failures` | 失败详情数组 |

---

## 脚本中的操作控制

在脚本中可以通过设置 `ctx.op` 控制每条文档的处理方式：

| `ctx.op` 值 | 行为 |
|-------------|------|
| `"index"` | 默认值，更新文档 |
| `"noop"` | 跳过该文档，不做任何写入 |
| `"delete"` | 删除该文档 |

```json
POST /website/_update_by_query
{
  "query": { "match_all": {} },
  "script": {
    "source": "if (ctx._source.views < 10) { ctx.op = 'noop' } else { ctx._source.popular = true }",
    "lang": "painless"
  }
}
```

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 单条文档更新 | [Update API]({{< relref "./update-api.md" >}}) |
| 按查询删除文档 | [Delete by Query]({{< relref "./delete-by-query.md" >}}) |
| 跨索引迁移数据 | [Reindex]({{< relref "/docs/operations/data-management/reindex.md" >}}) |
| 乐观并发控制 | [并发控制与版本]({{< relref "/docs/fundamentals/concurrency-and-versioning.md" >}}) |


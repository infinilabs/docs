---
title: "Update API"
date: 0001-01-01
description: "部分更新文档，支持 doc 合并、脚本更新、Upsert 模式。"
summary: "Update API #  基于现有文档做部分字段更新或脚本更新。 内部流程：读取当前 _source → 合并修改 → 写入新文档（不是原地修改）。
请求格式 #  POST /&lt;index&gt;/_update/&lt;_id&gt; 路径参数 #     参数 必需 说明     &lt;index&gt; 是 目标索引   &lt;_id&gt; 是 文档 ID    查询参数 #     参数 类型 默认值 说明     retry_on_conflict int 0 版本冲突时自动重试次数。适合&quot;计数器累加&quot;等顺序不敏感场景   routing string — 自定义路由   timeout time 1m 等待主分片可用的超时   refresh string false 更新后刷新策略：true、false、wait_for   _source string/boolean true 控制响应中是否返回 _source   _source_includes string — 响应中 _source 要包含的字段   _source_excludes string — 响应中 _source 要排除的字段   if_seq_no long — 乐观并发控制   if_primary_term long — 乐观并发控制   wait_for_active_shards string — 活跃分片数量   require_alias boolean false 要求目标必须是别名     注意：Update API 不支持 version / version_type 参数。如需乐观并发控制，请使用 if_seq_no + if_primary_term。"
---


# Update API

基于现有文档做部分字段更新或脚本更新。
内部流程：读取当前 `_source` → 合并修改 → 写入新文档（不是原地修改）。

## 请求格式

```
POST /<index>/_update/<_id>
```

## 路径参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `<index>` | **是** | 目标索引 |
| `<_id>` | **是** | 文档 ID |

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `retry_on_conflict` | int | `0` | 版本冲突时自动重试次数。适合"计数器累加"等顺序不敏感场景 |
| `routing` | string | — | 自定义路由 |
| `timeout` | time | `1m` | 等待主分片可用的超时 |
| `refresh` | string | `false` | 更新后刷新策略：`true`、`false`、`wait_for` |
| `_source` | string/boolean | `true` | 控制响应中是否返回 `_source` |
| `_source_includes` | string | — | 响应中 `_source` 要包含的字段 |
| `_source_excludes` | string | — | 响应中 `_source` 要排除的字段 |
| `if_seq_no` | long | — | 乐观并发控制 |
| `if_primary_term` | long | — | 乐观并发控制 |
| `wait_for_active_shards` | string | — | 活跃分片数量 |
| `require_alias` | boolean | `false` | 要求目标必须是别名 |

> **注意**：Update API **不支持** `version` / `version_type` 参数。如需乐观并发控制，请使用 `if_seq_no` + `if_primary_term`。

## 请求体字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `doc` | object | 要合并的部分文档。与现有 `_source` 做浅合并 |
| `upsert` | object | 当文档不存在时，按此内容创建文档 |
| `doc_as_upsert` | boolean | 为 `true` 时，若文档不存在则直接将 `doc` 作为新文档插入 |
| `script` | object/string | 使用脚本更新。可以是 inline 脚本字符串或 `{"source":"...","lang":"painless","params":{...}}` 对象 |
| `scripted_upsert` | boolean | 为 `true` 时，无论文档是否存在都执行脚本（结合 `upsert` 使用） |
| `detect_noop` | boolean | 为 `true` 时（默认），如果 `doc` 合并后没有实际变化，则不执行写入操作 |
| `_source` | object | 请求体内的 `_source` 过滤配置 |

## 示例

### 部分更新（doc merge）

给文档增加 `tags` 和 `views` 字段：

```json
POST /website/_update/1
{
  "doc": {
    "tags":  ["testing"],
    "views": 0
  }
}
```

已有字段保持不变，新字段被添加。

### 脚本更新

基于旧值做计算——将 `views` 加 1：

```json
POST /website/_update/1
{
  "script": {
    "source": "ctx._source.views += params.inc",
    "lang":   "painless",
    "params": { "inc": 1 }
  }
}
```

向数组追加元素：

```json
POST /website/_update/1
{
  "script": {
    "source": "ctx._source.tags.add(params.new_tag)",
    "lang":   "painless",
    "params": { "new_tag": "search" }
  }
}
```

> 脚本适合"简单算子 + 高价值更新"场景。复杂业务逻辑尽量放在上游服务完成。

### Upsert（不存在则插入）

计数器场景，第一次更新时文档可能还不存在：

```json
POST /website/_update/1
{
  "script": {
    "source": "ctx._source.views += 1",
    "lang":   "painless"
  },
  "upsert": {
    "views": 1
  }
}
```

- 第一次调用：文档不存在，按 `upsert` 内容创建，`views = 1`
- 后续调用：文档已存在，执行脚本，`views` 递增

### doc_as_upsert

"文档存在就合并 doc，不存在就把 doc 当新文档插入"：

```json
POST /website/_update/1
{
  "doc": {
    "title": "New title",
    "views": 0
  },
  "doc_as_upsert": true
}
```

### 冲突重试

顺序不敏感的更新（如页面浏览计数器），可以自动重试：

```json
POST /website/_update/1?retry_on_conflict=5
{
  "script": {
    "source": "ctx._source.views += 1",
    "lang":   "painless"
  }
}
```

最多自动重试 5 次版本冲突。

---

## 参考导航

| 需求 | 参见 |
|------|------|
| 按查询批量更新 | [Update by Query]({{< relref "./update-by-query.md" >}}) |
| 整文档覆盖写入 | [Index API]({{< relref "./index-api.md" >}}) |
| 乐观并发控制 | [并发控制与版本]({{< relref "/docs/fundamentals/concurrency-and-versioning.md" >}}) |


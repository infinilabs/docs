---
title: "分页与滚动"
date: 0001-01-01
description: "from/size 基础分页、Scroll 批量拉取、search_after 深分页、Point in Time (PIT) 的 API 与参数说明。"
summary: "分页与滚动 #  Easysearch 提供多种分页机制，适用于不同场景：
   方式 适用场景 一致性 深度限制     from/size 小规模翻页（前几页） 无（实时数据） 默认 10,000   search_after + sort 深分页、增量翻页 无（实时数据） 无   search_after + PIT 深分页、一致性翻页 快照一致 无   Scroll 批量导出、离线处理 快照一致 无    相关指南 #    分页与排序   from/size 基础分页 #  最基础的分页依赖 from 和 size 两个参数：
   参数 说明 默认值     from 起始偏移（0 基） 0   size 本页返回条数 10    GET shakespeare/_search { &#34;from&#34;: 0, &#34;size&#34;: 10, &#34;query&#34;: { &#34;match&#34;: { &#34;play_name&#34;: &#34;Hamlet&#34; } } } 计算页码对应的 from：from = size × (page_number - 1)"
---


# 分页与滚动

Easysearch 提供多种分页机制，适用于不同场景：

| 方式 | 适用场景 | 一致性 | 深度限制 |
|------|---------|--------|---------|
| from/size | 小规模翻页（前几页） | 无（实时数据） | 默认 10,000 |
| search_after + sort | 深分页、增量翻页 | 无（实时数据） | 无 |
| search_after + PIT | 深分页、一致性翻页 | 快照一致 | 无 |
| Scroll | 批量导出、离线处理 | 快照一致 | 无 |

## 相关指南

- [分页与排序]({{< relref "/docs/features/fulltext-search/pagination-and-sorting.md" >}})

---

## from/size 基础分页

最基础的分页依赖 `from` 和 `size` 两个参数：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `from` | 起始偏移（0 基） | 0 |
| `size` | 本页返回条数 | 10 |

```json
GET shakespeare/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "match": {
      "play_name": "Hamlet"
    }
  }
}
```

计算页码对应的 `from`：`from = size × (page_number - 1)`

也可以在 URL 参数中指定：

```
GET shakespeare/_search?from=0&size=10
```

### 注意事项

- **深度限制**：`from + size` 不能超过 `index.max_result_window`（默认 10,000），超过会报错
- **无状态**：每次请求看到的是"此刻最新"数据，翻页期间有新写入时可能看到重复或跳过的结果
- **分布式开销**：`from=10000, size=10` 实际上需要每个分片返回 10,010 条再合并，深页性能差

> **提示**：通过 `track_total_hits` 参数可以控制是否精确统计总命中数。设置为 `false` 可提升大结果集的分页性能：

```json
GET shakespeare/_search
{
  "track_total_hits": false,
  "from": 0,
  "size": 10,
  "query": {
    "match_all": {}
  }
}
```

也可以设置为具体数值（如 `1000`），只精确统计到该值，超过后显示为 `"gte"`。

---

## search_after 深分页

`search_after` 配合 `sort` 使用，通过上一页最后一条结果的排序值定位下一页，避免深分页性能问题。

### 基础用法

第一次查询需要指定 `sort`：

```json
GET shakespeare/_search
{
  "size": 10,
  "query": {
    "term": {
      "play_name": { "value": "Henry IV" }
    }
  },
  "sort": [
    { "line_id": { "order": "desc" } },
    { "speech_number": { "order": "desc" } }
  ]
}
```

每条结果会带一个 `sort` 数组，取最后一条的 `sort` 值作为下一页的起始点：

```json
GET shakespeare/_search
{
  "size": 10,
  "query": {
    "term": {
      "play_name": { "value": "Henry IV" }
    }
  },
  "sort": [
    { "line_id": { "order": "desc" } },
    { "speech_number": { "order": "desc" } }
  ],
  "search_after": [3202, 8]
}
```

> **注意**：`search_after` 数组的值数量和顺序必须与 `sort` 完全一致。

---

## Point in Time（PIT）

PIT 提供了一个轻量级的数据视图快照，配合 `search_after` 可以实现**一致性深分页**——在翻页过程中数据视图保持稳定，不受新写入影响。

### 创建 PIT

```json
POST /shakespeare/_pit?keep_alive=5m
```

返回一个 `pit_id`：

```json
{
  "id": "46ToAwMDaWR5BXV1aWQyK..."
}
```

### 使用 PIT + search_after 翻页

注意：使用 PIT 时，请求体中**不指定索引**（索引信息已绑定在 PIT 中）：

```json
GET _search
{
  "size": 10,
  "query": {
    "match": { "play_name": "Hamlet" }
  },
  "pit": {
    "id": "46ToAwMDaWR5BXV1aWQyK...",
    "keep_alive": "5m"
  },
  "sort": [
    { "line_id": { "order": "asc" } }
  ]
}
```

后续翻页使用 `search_after`：

```json
GET _search
{
  "size": 10,
  "query": {
    "match": { "play_name": "Hamlet" }
  },
  "pit": {
    "id": "46ToAwMDaWR5BXV1aWQyK...",
    "keep_alive": "5m"
  },
  "sort": [
    { "line_id": { "order": "asc" } }
  ],
  "search_after": [32700]
}
```

### 管理 PIT

```json
# 查看所有 PIT
GET _pit/_all

# 删除特定 PIT
DELETE _pit
{
  "pit_id": ["46ToAwMDaWR5BXV1aWQyK..."]
}

# 删除所有 PIT
DELETE _pit/_all
```

> **注意**：PIT 会占用资源（内存和段文件引用），用完后务必及时关闭。

---

## Scroll 搜索

`scroll` 适合"批量拉取大量结果"的场景（导出、离线分析等）。它会维护一个**搜索上下文快照**，数据视图是静态的，新写入不会影响遍历结果。

> **不推荐用于在线用户翻页**——Scroll 会长时间占用资源。在线翻页场景请使用 `search_after` 或 PIT。

### 发起 Scroll 查询

```json
GET shakespeare/_search?scroll=10m
{
  "size": 10000
}
```

返回一个 `_scroll_id`。

### 获取下一批

```json
GET _search/scroll
{
  "scroll": "10m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAUWdmpUZDhn..."
}
```

只要搜索上下文有效，就可以持续拉取。每次响应都会返回最新的 `scroll_id`，建议始终使用最新值。

### 并行 Scroll（Sliced Scroll）

对于超大结果集，可以用 **sliced scroll** 并行拉取，通过 `slice.id` 和 `slice.max` 切分：

```json
# 消费者 0
GET shakespeare/_search?scroll=10m
{
  "slice": { "id": 0, "max": 5 },
  "query": { "match_all": {} }
}

# 消费者 1
GET shakespeare/_search?scroll=10m
{
  "slice": { "id": 1, "max": 5 },
  "query": { "match_all": {} }
}
```

### 关闭 Scroll

```json
# 关闭特定 scroll
DELETE _search/scroll/DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAcW...

# 关闭所有 scroll
DELETE _search/scroll/_all
```

> **务必在用完后关闭**，否则直到超时前都会占用资源。


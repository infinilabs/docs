---
title: "Refresh、Flush 与 Force Merge"
date: 0001-01-01
description: "索引维护操作：手动刷新、持久化、段合并与缓存清理。"
summary: "Refresh、Flush 与 Force Merge #  这些是索引的日常维护操作，用于控制数据可见性、持久性和段文件结构。
Refresh：让新写入的文档可搜索 #  写入的文档不会立刻出现在搜索结果中——需要经过一次 refresh 操作，在内存中创建新的 Lucene 段（segment），文档才能被 _search 检索到。
 详见 写入与存储机制 了解 refresh 的工作原理。
 默认情况下，Easysearch 每 1 秒 自动执行一次 refresh（由 index.refresh_interval 控制）。但你也可以手动触发：
// 刷新特定索引 POST /my-index/_refresh // 刷新多个索引 POST /index-1,index-2/_refresh // 刷新所有索引 POST /_refresh 响应：
{ &#34;_shards&#34;: { &#34;total&#34;: 10, &#34;successful&#34;: 5, &#34;failed&#34;: 0 } } 查询参数 #     参数 类型 默认值 说明     expand_wildcards String open 通配符展开策略   ignore_unavailable Boolean false 忽略不存在的索引   allow_no_indices Boolean true 通配符未匹配时是否报错    常见场景 #   批量写入后立即搜索：导入数据后手动 refresh 确保数据可见 测试/CI 环境：写入后立即 refresh 以验证结果   生产环境注意：频繁手动 refresh 会增加小段数量，影响搜索性能。大批量写入时建议临时关闭自动 refresh（设为 &quot;-1&quot;），导入完成后再恢复。"
---


# Refresh、Flush 与 Force Merge

这些是索引的日常维护操作，用于控制数据可见性、持久性和段文件结构。

## Refresh：让新写入的文档可搜索

写入的文档不会立刻出现在搜索结果中——需要经过一次 **refresh** 操作，在内存中创建新的 Lucene 段（segment），文档才能被 `_search` 检索到。

> 详见 [写入与存储机制]({{< relref "/docs/fundamentals/write-and-storage" >}}) 了解 refresh 的工作原理。

默认情况下，Easysearch 每 **1 秒** 自动执行一次 refresh（由 `index.refresh_interval` 控制）。但你也可以手动触发：

```json
// 刷新特定索引
POST /my-index/_refresh

// 刷新多个索引
POST /index-1,index-2/_refresh

// 刷新所有索引
POST /_refresh
```

响应：

```json
{
  "_shards": {
    "total": 10,
    "successful": 5,
    "failed": 0
  }
}
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配时是否报错 |

### 常见场景

- **批量写入后立即搜索**：导入数据后手动 refresh 确保数据可见
- **测试/CI 环境**：写入后立即 refresh 以验证结果

> **生产环境注意**：频繁手动 refresh 会增加小段数量，影响搜索性能。大批量写入时建议临时关闭自动 refresh（设为 `"-1"`），导入完成后再恢复。

## Flush：将数据持久化到磁盘

Flush 触发 Lucene 的 **commit** 操作，将内存中的段写入磁盘，并清空事务日志（translog）。Flush 之后，即使节点崩溃也不会丢失数据。

> Easysearch 会自动管理 flush（当 translog 达到 `index.translog.flush_threshold_size` 时触发），通常**不需要手动 flush**。

```json
// Flush 特定索引
POST /my-index/_flush

// Flush 所有索引
POST /_flush
```

响应：

```json
{
  "_shards": {
    "total": 10,
    "successful": 5,
    "failed": 0
  }
}
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `force` | Boolean | `false` | 即使没有需要提交的变更也强制 flush |
| `wait_if_ongoing` | Boolean | `true` | 如果另一个 flush 正在进行，是否等待而不是立即返回 |
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配时是否报错 |

### 何时手动 Flush

- 在执行**滚动重启**前，手动 flush 所有索引可以加速节点恢复（减少需要重放的 translog）
- 在关闭索引之前

```json
// 滚动重启前：flush 所有索引
POST /_flush
```

## Force Merge：合并段文件

段合并（merge）会在后台自动进行，但有时你可能需要手动触发强制合并：

- 将已删除文档从段中清除，释放磁盘空间
- 将小段合并为大段，提高搜索效率
- 将只读索引合并为一个段，获得最佳查询性能

```json
// 将索引合并到最多 1 个段
POST /my-index/_forcemerge?max_num_segments=1

// 仅清除已删除文档
POST /my-index/_forcemerge?only_expunge_deletes=true

// 对所有索引执行
POST /_forcemerge?max_num_segments=1
```

响应：

```json
{
  "_shards": {
    "total": 10,
    "successful": 5,
    "failed": 0
  }
}
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `max_num_segments` | Integer | `-1`（无限制） | 合并后的最大段数。设为 `1` 表示完全合并 |
| `only_expunge_deletes` | Boolean | `false` | 仅合并包含已删除文档的段 |
| `flush` | Boolean | `true` | 合并完成后是否自动 flush |
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配时是否报错 |

### 使用建议

| 场景 | 推荐操作 |
|------|----------|
| 只读/归档索引 | `max_num_segments=1`，获得最佳搜索性能 |
| 大量删除后回收空间 | `only_expunge_deletes=true` |
| 仍在写入的索引 | **不要** force merge——后台自动合并足够 |

> ⚠️ **对正在写入的索引执行 force merge 会导致很大的段产生，后续写入又会产生小段，反而加重合并压力。**

## 清除缓存

清除索引级别的缓存（查询缓存、请求缓存、字段数据缓存），释放内存。

```json
// 清除特定索引的所有缓存
POST /my-index/_cache/clear

// 清除所有索引的缓存
POST /_cache/clear

// 只清除特定类型的缓存
POST /my-index/_cache/clear?query=true
POST /my-index/_cache/clear?request=true
POST /my-index/_cache/clear?fielddata=true

// 清除特定字段的字段数据缓存
POST /my-index/_cache/clear?fields=field1,field2
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `query` | Boolean | — | 是否清除查询缓存 |
| `request` | Boolean | — | 是否清除请求缓存 |
| `fielddata` | Boolean | — | 是否清除字段数据缓存 |
| `fields` | String | — | 逗号分隔的字段名，指定清除哪些字段的缓存 |
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配时是否报错 |

## 操作对比

| 操作 | 作用 | 性能影响 | 需要手动触发？ |
|------|------|----------|---------------|
| Refresh | 新文档变得可搜索 | 创建新段，增加段数 | 一般不需要 |
| Flush | 段持久化到磁盘，清空 translog | 磁盘 IO | 一般不需要 |
| Force Merge | 合并段、清除已删除文档 | CPU + IO 密集 | 只在只读索引上 |
| Cache Clear | 释放内存中的缓存 | 后续查询需要重建缓存 | 内存压力时 |

## 下一步

- [索引设置](./index-settings.md)：`refresh_interval`、`translog.durability`、合并策略等
- [索引统计与监控](./index-stats.md)：查看段数量、合并状态等指标
- [写入与存储机制]({{< relref "/docs/fundamentals/write-and-storage" >}})：Refresh、Flush 的底层原理以及为什么搜索不是实时的


---
title: "索引统计与监控"
date: 0001-01-01
description: "查看索引的统计信息、段详情、恢复进度和分片存储状态。"
summary: "索引统计与监控 #  Easysearch 提供了一组 API，用于查看索引的运行状态、性能指标和存储详情。这些信息是性能调优和故障排查的基础。
索引统计（Index Stats） #  获取索引级别的统计信息，包括文档数、存储大小、写入/查询/合并等操作的计数和耗时。
// 获取特定索引的所有统计 GET /my-index/_stats // 获取所有索引的统计 GET /_stats // 只获取特定指标 GET /my-index/_stats/docs,store // 多索引 GET /index-1,index-2/_stats/indexing,search 可用指标 #     指标 说明     docs 文档数和已删除文档数   store 索引存储大小   indexing 写入操作统计（总数、耗时、当前进行中等）   get Get 操作统计   search 搜索操作统计（query、fetch 阶段）   merge 段合并统计（次数、耗时、大小）   refresh Refresh 统计   flush Flush 统计   query_cache 查询缓存命中率和大小   fielddata Fielddata 内存使用   completion Completion Suggester 内存使用   segments 段数量、内存占用   translog Translog 大小和操作数   request_cache 请求缓存命中率   recovery 恢复操作统计   warmer Warmer 统计   _all 所有指标（默认）    查询参数 #     参数 类型 默认值 说明     level String indices 聚合级别：cluster、indices、shards   fields String — 逗号分隔的字段名（用于 completion 和 fielddata 指标）   completion_fields String — Completion 字段名   fielddata_fields String — Fielddata 字段名   include_segment_file_sizes Boolean false 在 segments 指标中包含文件大小详情   include_unloaded_segments Boolean false 包含未加载的段信息   forbid_closed_indices Boolean true 禁止查询已关闭索引的统计   expand_wildcards String open 通配符展开策略   ignore_unavailable Boolean false 忽略不存在的索引    响应示例（节选） #  { &#34;_all&#34;: { &#34;primaries&#34;: { &#34;docs&#34;: { &#34;count&#34;: 1500000, &#34;deleted&#34;: 2500 }, &#34;store&#34;: { &#34;size_in_bytes&#34;: 1073741824 }, &#34;indexing&#34;: { &#34;index_total&#34;: 1500000, &#34;index_time_in_millis&#34;: 45000, &#34;index_current&#34;: 0 }, &#34;search&#34;: { &#34;query_total&#34;: 350000, &#34;query_time_in_millis&#34;: 12000, &#34;query_current&#34;: 2 }, &#34;segments&#34;: { &#34;count&#34;: 42, &#34;memory_in_bytes&#34;: 52428800 } } } } 常见用法 #  // 查看段数量（判断是否需要 force merge） GET /my-index/_stats/segments // 查看写入速率 GET /my-index/_stats/indexing // 查看查询缓存命中率 GET /my-index/_stats/query_cache,request_cache // 按分片级别查看统计 GET /my-index/_stats?"
---


# 索引统计与监控

Easysearch 提供了一组 API，用于查看索引的运行状态、性能指标和存储详情。这些信息是性能调优和故障排查的基础。

## 索引统计（Index Stats）

获取索引级别的统计信息，包括文档数、存储大小、写入/查询/合并等操作的计数和耗时。

```json
// 获取特定索引的所有统计
GET /my-index/_stats

// 获取所有索引的统计
GET /_stats

// 只获取特定指标
GET /my-index/_stats/docs,store

// 多索引
GET /index-1,index-2/_stats/indexing,search
```

### 可用指标

| 指标 | 说明 |
|------|------|
| `docs` | 文档数和已删除文档数 |
| `store` | 索引存储大小 |
| `indexing` | 写入操作统计（总数、耗时、当前进行中等） |
| `get` | Get 操作统计 |
| `search` | 搜索操作统计（query、fetch 阶段） |
| `merge` | 段合并统计（次数、耗时、大小） |
| `refresh` | Refresh 统计 |
| `flush` | Flush 统计 |
| `query_cache` | 查询缓存命中率和大小 |
| `fielddata` | Fielddata 内存使用 |
| `completion` | Completion Suggester 内存使用 |
| `segments` | 段数量、内存占用 |
| `translog` | Translog 大小和操作数 |
| `request_cache` | 请求缓存命中率 |
| `recovery` | 恢复操作统计 |
| `warmer` | Warmer 统计 |
| `_all` | 所有指标（默认） |

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `level` | String | `indices` | 聚合级别：`cluster`、`indices`、`shards` |
| `fields` | String | — | 逗号分隔的字段名（用于 `completion` 和 `fielddata` 指标） |
| `completion_fields` | String | — | Completion 字段名 |
| `fielddata_fields` | String | — | Fielddata 字段名 |
| `include_segment_file_sizes` | Boolean | `false` | 在 `segments` 指标中包含文件大小详情 |
| `include_unloaded_segments` | Boolean | `false` | 包含未加载的段信息 |
| `forbid_closed_indices` | Boolean | `true` | 禁止查询已关闭索引的统计 |
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 忽略不存在的索引 |

### 响应示例（节选）

```json
{
  "_all": {
    "primaries": {
      "docs": {
        "count": 1500000,
        "deleted": 2500
      },
      "store": {
        "size_in_bytes": 1073741824
      },
      "indexing": {
        "index_total": 1500000,
        "index_time_in_millis": 45000,
        "index_current": 0
      },
      "search": {
        "query_total": 350000,
        "query_time_in_millis": 12000,
        "query_current": 2
      },
      "segments": {
        "count": 42,
        "memory_in_bytes": 52428800
      }
    }
  }
}
```

### 常见用法

```json
// 查看段数量（判断是否需要 force merge）
GET /my-index/_stats/segments

// 查看写入速率
GET /my-index/_stats/indexing

// 查看查询缓存命中率
GET /my-index/_stats/query_cache,request_cache

// 按分片级别查看统计
GET /my-index/_stats?level=shards
```

## 索引段信息（Segments）

查看索引中每个分片的段（segment）详情，包括段名、文档数、大小、是否已合并等。

```json
// 特定索引
GET /my-index/_segments

// 所有索引
GET /_segments
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `verbose` | Boolean | `false` | 显示详细的段信息 |
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配时是否报错 |

### 响应示例（节选）

```json
{
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "indices": {
    "my-index": {
      "shards": {
        "0": [
          {
            "routing": {
              "state": "STARTED",
              "primary": true,
              "node": "node-1"
            },
            "num_committed_segments": 5,
            "num_search_segments": 5,
            "segments": {
              "_0": {
                "generation": 0,
                "num_docs": 100000,
                "deleted_docs": 0,
                "size_in_bytes": 52428800,
                "committed": true,
                "search": true,
                "version": "8.11.0",
                "compound": true
              }
            }
          }
        ]
      }
    }
  }
}
```

段信息的关键字段：

| 字段 | 说明 |
|------|------|
| `num_docs` | 段中的文档数（不含已删除） |
| `deleted_docs` | 已删除但未合并清理的文档数 |
| `size_in_bytes` | 段文件大小 |
| `committed` | 是否已提交到磁盘 |
| `compound` | 是否使用复合文件格式 |

## 恢复信息（Recovery）

查看索引分片的恢复进度，包括初始恢复、副本复制和迁移。

```json
// 特定索引
GET /my-index/_recovery

// 所有索引
GET /_recovery

// 仅显示进行中的恢复
GET /my-index/_recovery?active_only=true
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `detailed` | Boolean | `false` | 显示详细恢复信息（包含文件列表） |
| `active_only` | Boolean | `false` | 仅显示正在进行中的恢复 |
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 忽略不存在的索引 |

### 恢复类型

| 类型 | 说明 |
|------|------|
| `STORE` | 从本地磁盘恢复（节点重启） |
| `SNAPSHOT` | 从快照恢复 |
| `REPLICA` | 从主分片复制到副本 |
| `RELOCATING` | 分片迁移到另一个节点 |

### 响应示例（节选）

```json
{
  "my-index": {
    "shards": [
      {
        "id": 0,
        "type": "STORE",
        "stage": "DONE",
        "primary": true,
        "start_time_in_millis": 1609459200000,
        "stop_time_in_millis": 1609459205000,
        "total_time_in_millis": 5000,
        "source": {},
        "target": {
          "id": "node-1-id",
          "host": "192.168.1.1",
          "name": "node-1"
        },
        "index": {
          "size": {
            "total_in_bytes": 1073741824,
            "recovered_in_bytes": 1073741824,
            "percent": "100.0%"
          },
          "files": {
            "total": 42,
            "recovered": 42,
            "percent": "100.0%"
          }
        },
        "translog": {
          "recovered": 0,
          "total": 0,
          "percent": "100.0%",
          "total_on_start": 0
        }
      }
    ]
  }
}
```

## 分片存储状态（Shard Stores）

查看每个分片副本的存储状态和分配情况，用于诊断未分配分片的问题。

```json
// 特定索引
GET /my-index/_shard_stores

// 所有索引
GET /_shard_stores

// 只查看未分配的分片存储状态
GET /_shard_stores?status=red
```

## 使用 Cat API 快速查看

对于日常巡检，Cat API 提供更简洁的表格输出：

```json
// 索引列表与大小
GET _cat/indices?v&s=store.size:desc

// 特定索引的分片分布
GET _cat/shards/my-index?v

// 段信息
GET _cat/segments/my-index?v

// 恢复进度
GET _cat/recovery/my-index?v&active_only=true
```

## 监控检查清单

| 检查项 | API | 关注指标 |
|--------|-----|----------|
| 段数量是否过多 | `_stats/segments` | `segments.count`，只读索引建议合并到 1 |
| 已删除文档占比 | `_stats/docs` | `deleted / (count + deleted)`，过高应 force merge |
| 查询缓存命中率 | `_stats/query_cache` | `hit_count / (hit_count + miss_count)` |
| 写入速率 | `_stats/indexing` | `index_total` 增量 / 时间窗口 |
| 恢复是否完成 | `_recovery?active_only=true` | 无结果 = 恢复完成 |
| 磁盘空间 | `_cat/indices?v&h=index,store.size` | 定期检查增长趋势 |

## 下一步

- [Refresh、Flush 与 Force Merge](./refresh-flush-forcemerge.md)：段维护操作
- [索引设置](./index-settings.md)：合并策略、刷新间隔等调优参数
- [监控]({{< relref "/docs/operations/monitoring" >}})：集群级别的监控
- [故障排查]({{< relref "/docs/operations/troubleshooting" >}})：常见问题诊断


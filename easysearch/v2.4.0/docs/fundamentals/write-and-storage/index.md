---
title: "写入与存储机制"
date: 0001-01-01
description: "文档从写入请求到可被搜索再到持久化落盘的全过程：内存缓冲、refresh、translog、flush 与段合并。"
summary: "写入与存储机制 #  当你向 Easysearch 索引一个文档时，数据要经历多个阶段才能最终安全地持久化到磁盘。理解这个过程，有助于你在性能调优和故障恢复时做出正确决策。
写入的全景图 #  一个文档从写入请求到可被搜索再到持久化落盘，大致经历以下阶段：
客户端请求 ↓ 协调节点路由到主分片 ↓ 主分片写入： 1. 追加到 translog（事务日志） 2. 写入内存索引缓冲区（in-memory buffer） ↓ refresh（默认每秒）: - 缓冲区内容写入新的段（segment）到文件系统缓存 - 新段可被搜索（近实时） - 缓冲区清空，translog 保留 ↓ flush（translog 超过 512MB 阈值时触发）: - 段从文件系统缓存 fsync 到磁盘 - 写入提交点（commit point） - translog 清空 ↓ 段合并（后台持续）: - 小段合并为大段 - 清理已删除文档 - 减少段数量，提升搜索性能 下面按阶段逐一展开。
近实时搜索：为什么不是&quot;实时&quot; #  Easysearch 被称为 近实时（Near Real-Time, NRT） 搜索引擎——文档写入后并非立刻可搜索，但通常在一秒内就能被检索到。
背后的原因在于：磁盘 I/O 是瓶颈。提交（commit）一个新的段到磁盘需要 fsync，开销非常大。如果每索引一个文档就执行一次 fsync，性能将无法接受。
Easysearch 的做法是：将新段先写入文件系统缓存（OS page cache）——这一步代价很低；稍后再 fsync 到磁盘。只要新段进入了文件系统缓存，就可以像其他文件一样被打开和读取，从而对搜索可见。"
---


# 写入与存储机制

当你向 Easysearch 索引一个文档时，数据要经历多个阶段才能最终安全地持久化到磁盘。理解这个过程，有助于你在性能调优和故障恢复时做出正确决策。

## 写入的全景图

一个文档从写入请求到可被搜索再到持久化落盘，大致经历以下阶段：

```text
客户端请求
    ↓
协调节点路由到主分片
    ↓
主分片写入：
  1. 追加到 translog（事务日志）
  2. 写入内存索引缓冲区（in-memory buffer）
    ↓
refresh（默认每秒）:
  - 缓冲区内容写入新的段（segment）到文件系统缓存
  - 新段可被搜索（近实时）
  - 缓冲区清空，translog 保留
    ↓
flush（translog 超过 512MB 阈值时触发）:
  - 段从文件系统缓存 fsync 到磁盘
  - 写入提交点（commit point）
  - translog 清空
    ↓
段合并（后台持续）:
  - 小段合并为大段
  - 清理已删除文档
  - 减少段数量，提升搜索性能
```

下面按阶段逐一展开。

## 近实时搜索：为什么不是"实时"

Easysearch 被称为 **近实时（Near Real-Time, NRT）** 搜索引擎——文档写入后并非立刻可搜索，但通常在一秒内就能被检索到。

背后的原因在于：磁盘 I/O 是瓶颈。提交（commit）一个新的段到磁盘需要 `fsync`，开销非常大。如果每索引一个文档就执行一次 `fsync`，性能将无法接受。

Easysearch 的做法是：将新段先写入**文件系统缓存**（OS page cache）——这一步代价很低；稍后再 `fsync` 到磁盘。只要新段进入了文件系统缓存，就可以像其他文件一样被打开和读取，从而对搜索可见。

这就是"近实时"的由来：文档写入后最多延迟一个 refresh 周期（默认 1 秒）即可被搜索到。

## Translog：写入安全网

每次写入操作首先追加到 **translog（事务日志）**。即使在 refresh 和 flush 之间发生崩溃，重启后 Easysearch 会重放 translog 恢复数据。

- 默认每次写请求完成后 `fsync` translog 到磁盘
- 在主分片和副本分片上都会写入 translog
- 客户端收到 `200 OK` 时，translog 已经持久化

translog 还提供实时的 CRUD：当你通过 ID 查询、更新、删除一个文档时，Easysearch 会先检查 translog 中是否有最近的变更，再去段里检索。这意味着它总是能获取到文档的最新版本。

## 内存缓冲区：等待刷新

文档同时被写入内存索引缓冲区（in-memory buffer）。此时文档**还不可搜索**，需要等待下一次 refresh。

## Refresh：让文档可搜索

在 Easysearch 中，写入和打开一个新段的轻量过程叫做 **refresh**。默认情况下每个分片每秒自动刷新一次。

执行流程：

1. 将缓冲区内容写入新的段到**文件系统缓存**（不是磁盘）
2. 新段被打开，文档变为可搜索
3. 缓冲区清空，translog 保留

### refresh API

可以手动触发 refresh：

```json
POST /_refresh          // 刷新所有索引
POST /blogs/_refresh    // 只刷新 blogs 索引
```

> 手动 refresh 在写测试时很有用，但不要在生产环境每次索引文档后都手动 refresh。你的应用需要理解并接受 Easysearch 的近实时特性。

### 调整刷新频率

并非所有场景都需要每秒刷新。批量导入或日志场景可以降低刷新频率以提升写入性能：

```json
PUT /my_logs
{
  "settings": {
    "refresh_interval": "30s"
  }
}
```

`refresh_interval` 支持动态更新。在大批量导入时，可以先关闭自动刷新，导入完成后再恢复：

```json
PUT /my_logs/_settings
{ "refresh_interval": -1 }      // 关闭自动刷新

PUT /my_logs/_settings
{ "refresh_interval": "1s" }    // 恢复每秒刷新
```

> ⚠️ `refresh_interval` 的值需要带时间单位，例如 `1s`（1 秒）或 `2m`（2 分钟）。绝对值 `1` 表示 1 毫秒——会让集群陷入瘫痪。

## Flush：真正的持久化

Flush 将段从文件系统缓存 `fsync` 到物理磁盘，执行一次完整的提交（commit）：

1. 所有在内存缓冲区的文档被写入一个新的段
2. 缓冲区被清空
3. 提交点（commit point）被写入磁盘，记录当前所有活跃的段
4. 文件系统缓存通过 `fsync` 被刷新到磁盘
5. 旧的 translog 被删除，创建新的 translog

Flush 默认在 translog 大小超过 `512MB`（`index.translog.flush_threshold_size`）时自动触发。Easysearch **没有**基于时间的周期性 flush 定时器——flush 完全由 translog 大小驱动。

### flush API

```json
POST /blogs/_flush
POST /_flush?wait_for_ongoing
```

通常不需要手动 flush。但在重启节点或关闭索引之前执行 flush 可以加速恢复——因为 translog 越短，恢复越快。

### Translog 安全性

默认情况下 translog 在每次写请求完成之后执行 `fsync`（在主分片和副本分片都会发生）。对于大容量且偶尔丢失几秒数据也不严重的集群，可以使用异步 fsync：

```json
PUT /my_index/_settings
{
    "index.translog.durability": "async",
    "index.translog.sync_interval": "5s"
}
```

> ⚠️ 异步 translog 意味着在发生 crash 时，可能丢失 `sync_interval` 时间段的数据。如果不确定后果，请使用默认的 `"index.translog.durability": "request"`。

## 段合并：后台优化

由于每次 refresh 都会创建一个新的段，段数量会快速增长。而段数目太多会带来问题：每个段都消耗文件句柄、内存和 CPU 资源，更重要的是每次搜索都要轮流检查每个段，段越多搜索越慢。

Easysearch 通过在后台进行**段合并（merge）**来解决这个问题：

- 小段被合并到大段，大段再被合并到更大的段
- 已标记删除的文档在合并时被真正清理
- 合并过程中不会中断索引和搜索
- Easysearch 默认对合并流程进行资源限制，保证搜索性能不受影响

### 强制合并（Force Merge）

对于不再更新的只读索引（例如按天滚动的历史日志索引），可以使用强制合并将每个分片合并为一个单独的段：

```json
POST /logstash-2024-10/_forcemerge?max_num_segments=1
```

> ⚠️ 不要在活跃更新的索引上执行强制合并——后台合并流程已经能很好地完成工作。强制合并不受资源限制，可能消耗大量 I/O。

## 写入操作类型

| 操作 | 说明 | 内部行为 |
|------|------|----------|
| `PUT /index/_doc/id` | 索引（创建或覆盖） | 如果文档已存在，旧版本标记删除，新版本写入新段 |
| `POST /index/_doc` | 自动生成 ID 的索引 | 与上面相同，ID 由 Easysearch 生成 |
| `PUT /index/_doc/id/_create` | 仅创建（不覆盖） | 如果 ID 已存在，返回 409 Conflict |
| `POST /index/_update/id` | 部分更新 | 内部：读取 → 合并 → 重新索引整个文档 |
| `DELETE /index/_doc/id` | 删除 | 标记删除，段合并时真正清理 |
| `POST /index/_bulk` | 批量操作 | 按分片拆分并行执行，换行符分隔格式 |

## 文档的"不可变"本质

Easysearch 中有一个关键事实：

> **文档一旦写入段，就不会被修改。更新 = 标记旧版本删除 + 写入新版本。删除 = 标记删除 + 段合并时真正清理。**

这种不可变性带来了并发安全、缓存友好和压缩优势，但也意味着频繁更新会产生大量需要合并的小段。

## 性能调优要点

| 场景 | 建议 |
|------|------|
| 批量导入 | 关闭 refresh（`-1`），增大 bulk 批次（5-15MB），导入完成后恢复 |
| 日志场景 | 增大 `refresh_interval`（如 `30s`），减少段创建频率 |
| 高可靠性 | 保持 translog 默认同步模式（`request`） |
| 容忍少量丢失 | 可用异步 translog（`async`，5s 间隔） |
| 只读索引 | 执行 `_forcemerge?max_num_segments=1` 合并为单段 |

## 小结

| 阶段 | 作用 | 默认频率 |
|------|------|----------|
| Translog | 写入安全保障，崩溃恢复 | 每次写请求 |
| 内存缓冲 | 暂存文档，等待 refresh | — |
| Refresh | 让文档可搜索（近实时） | 每秒 |
| Flush | 将段持久化到磁盘 | translog 超过 512MB 时 |
| 段合并 | 减少段数量，清理删除 | 后台持续 |

- 写入先走 translog 保证安全，再走内存缓冲等待 refresh
- Refresh 使文档可搜索（近实时），flush 使数据持久化到磁盘
- 更新和删除不是原地修改，而是"标记删除 + 写新版本"
- 段合并在后台自动清理删除标记、减少段数量

下一步可以继续阅读：

- [Lucene 底层原理]({{< relref "./lucene-internals.md" >}})：倒排索引、段结构、文件格式
- [分布式写入过程]({{< relref "./distributed-write.md" >}})：文档如何在分片间路由和复制
- [文档操作]({{< relref "/docs/features/document-operations/_index.md" >}})：API 使用详情

### 最佳实践

- [数据生命周期与保留策略]({{< relref "/docs/best-practices/data-lifecycle.md" >}})：冷热分层、自动清理、容量规划
- [索引与分片设计]({{< relref "/docs/best-practices/index-design.md" >}})：分片规划、写入性能优化


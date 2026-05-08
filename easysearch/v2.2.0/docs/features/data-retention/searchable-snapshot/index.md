---
title: "可搜索快照"
date: 0001-01-01
description: "直接在对象存储上的快照数据中搜索，无需完整恢复索引，让冷数据也能被随时查询。"
summary: "可搜索快照（Searchable Snapshots） #  Easysearch 快照搜索允许系统直接在对象存储（如 S3、MinIO、OSS）上挂载快照索引，让归档数据瞬间具备检索能力——无需将快照恢复为完整索引，即使是 PB 级快照数据也能在数分钟内完成挂载并开放搜索。
 核心优势 #  分钟级上线，零恢复等待 #  传统快照恢复需要将全量数据拷贝到本地磁盘，耗时可能达数小时。快照搜索消除了这一步骤，仅挂载元数据，数据按需从对象存储拉取。
极低的存储成本 #  核心数据保留在低成本的对象存储中，仅在查询时按需调用，大幅节省高性能磁盘（SSD）的占用空间。
统一的搜索体验 #  快照搜索与在线集群共享统一的 API 和查询语法，业务人员无需学习新接口即可调取历史记录。
 前置条件 #  配置 search 角色节点 #  只有角色为 search 的节点才能执行快照搜索。在 easysearch.yml 中配置：
node.name: search-node node.roles: [search] Docker 环境：
- node.roles: [search]  集群中至少需要一个 search 角色节点才能使用可搜索快照功能。
 注册快照仓库 #  以 MinIO（S3 兼容）为例：
PUT _snapshot/my-repo { &#34;type&#34;: &#34;s3&#34;, &#34;settings&#34;: { &#34;access_key&#34;: &#34;minioadmin&#34;, &#34;secret_key&#34;: &#34;minioadmin&#34;, &#34;bucket&#34;: &#34;es-bucket&#34;, &#34;endpoint&#34;: &#34;http://127."
---


# 可搜索快照（Searchable Snapshots）

Easysearch 快照搜索允许系统直接在对象存储（如 S3、MinIO、OSS）上挂载快照索引，让归档数据瞬间具备检索能力——无需将快照恢复为完整索引，即使是 PB 级快照数据也能在数分钟内完成挂载并开放搜索。

---

## 核心优势

### 分钟级上线，零恢复等待

传统快照恢复需要将全量数据拷贝到本地磁盘，耗时可能达数小时。快照搜索消除了这一步骤，仅挂载元数据，数据按需从对象存储拉取。

### 极低的存储成本

核心数据保留在低成本的对象存储中，仅在查询时按需调用，大幅节省高性能磁盘（SSD）的占用空间。

### 统一的搜索体验

快照搜索与在线集群共享统一的 API 和查询语法，业务人员无需学习新接口即可调取历史记录。

---

## 前置条件

### 配置 search 角色节点

只有角色为 `search` 的节点才能执行快照搜索。在 `easysearch.yml` 中配置：

```yaml
node.name: search-node
node.roles: [search]
```

Docker 环境：

```yaml
- node.roles: [search]
```

> 集群中至少需要一个 `search` 角色节点才能使用可搜索快照功能。

### 注册快照仓库

以 MinIO（S3 兼容）为例：

```json
PUT _snapshot/my-repo
{
  "type": "s3",
  "settings": {
    "access_key": "minioadmin",
    "secret_key": "minioadmin",
    "bucket": "es-bucket",
    "endpoint": "http://127.0.0.1:9000",
    "compress": true
  }
}
```

---

## 使用方法

### 1. 创建快照（如果尚未创建）

```json
PUT _snapshot/my-repo/snapshot-2024
{
  "indices": "logs-2024-*",
  "include_global_state": false
}
```

### 2. 挂载为可搜索快照索引

使用 `_restore` API 并指定 `remote_snapshot` 存储类型：

```json
POST _snapshot/my-repo/snapshot-2024/_restore
{
  "indices": "logs-2024-01",
  "rename_pattern": "(.+)",
  "rename_replacement": "searchable-$1",
  "storage_type": "remote_snapshot"
}
```

#### 存储类型说明

| 类型 | 说明 |
|------|------|
| `local` | 所有快照数据下载到本地存储（传统恢复） |
| `remote_snapshot` | 仅下载元数据，数据按需从远程仓库拉取 |

#### 恢复参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `indices` | `string` | 要恢复的索引名称（支持通配符） |
| `rename_pattern` | `string` | 索引名称重命名的正则匹配模式 |
| `rename_replacement` | `string` | 索引名称重命名的替换字符串 |
| `storage_type` | `string` | 存储类型：`local` 或 `remote_snapshot` |
| `ignore_unavailable` | `boolean` | 是否忽略不存在的索引 |
| `include_global_state` | `boolean` | 是否包含集群全局状态 |
| `include_aliases` | `boolean` | 是否恢复别名 |
| `partial` | `boolean` | 是否允许部分恢复 |

### 3. 搜索快照数据

像查询普通索引一样搜索：

```json
POST /searchable-logs-2024-01/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "2024-01-01",
        "lte": "2024-01-31"
      }
    }
  }
}
```

支持完整的查询语法：过滤、排序、聚合等。

---

## 与异步搜索结合

快照搜索通常面向大时间跨度与大数据量场景，I/O 延迟可能较高。推荐与 [异步搜索]({{< relref "/docs/features/async-search" >}}) 结合使用：

```json
POST /searchable-logs-2024-*/_async_search
{
  "size": 0,
  "query": {
    "match": { "message": "OutOfMemoryError" }
  },
  "aggs": {
    "monthly_errors": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      }
    }
  }
}
```

---

## 冷热数据分层架构

快照搜索是实现数据分层存储的关键组件：

```
┌─────────────┐     ┌─────────────┐     ┌──────────────────┐
│  热数据 Hot  │ ──→ │ 温数据 Warm  │ ──→ │  冷数据 Cold      │
│  SSD 高性能  │     │  HDD 大容量  │     │  对象存储（S3）    │
│  实时读写    │     │  只读/低频   │     │  快照搜索按需访问   │
└─────────────┘     └─────────────┘     └──────────────────┘
```

- **热数据**：高性能集群实时访问
- **温数据**：降低副本数、只读，仍在集群中
- **冷数据**：低成本对象存储 + 快照搜索按需查询

> 配合 [索引生命周期管理（ILM）]({{< relref "/docs/features/data-retention/ilm" >}})，可以自动化实现数据在各层之间的流转。

---

## 应用场景

| 场景 | 说明 |
|------|------|
| **历史日志与审计分析** | 安全审计、合规检查、问题回溯，无需恢复历史索引 |
| **长周期数据留存** | 满足监管或业务对数据长期保存的要求，同时控制成本 |
| **冷热数据分层** | 在线集群只保留热数据，历史数据迁移至快照存储 |
| **低频但高价值查询** | 大跨度、低频次、分析型查询，不影响在线系统性能 |

---

## 缓存机制

可搜索快照索引采用智能缓存策略：

- **频繁访问的段**会被缓存到 search 节点的本地磁盘
- **最近最少使用（LRU）** 策略自动淘汰不常访问的段
- 缓存数据与节点的通用索引共享磁盘空间

> **建议**：为 `search` 角色节点配置充足的本地磁盘空间，以提升缓存命中率和查询性能。

---

## 注意事项

- 快照搜索索引是**只读**的，任何写入操作都会报错
- 查询性能取决于对象存储的 I/O 性能和网络带宽
- 首次查询可能较慢（冷启动），后续查询随缓存预热会加速
- 集群中至少需要一个 `search` 角色节点
- 对象存储的访问成本（API 调用次数）需要纳入运营预算考量

---

## 相关文档

- [备份与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}})
- [异步搜索]({{< relref "/docs/features/async-search" >}})
- [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}})


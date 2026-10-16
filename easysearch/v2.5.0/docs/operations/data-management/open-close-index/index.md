---
title: "开关索引与索引限制"
date: 0001-01-01
description: "关闭/打开索引以节省资源，以及使用 Index Block 限制索引的读写行为。"
summary: "开关索引与索引限制 #  关闭索引 #  关闭的索引不消耗集群资源（CPU、内存、文件句柄），但索引数据仍保留在磁盘上。关闭后索引不能读写，但可以随时重新打开。
典型场景：
 历史索引暂时不需要查询，但不想删除 修改 Static 类型的索引设置（必须先关闭索引） 降低集群负载  POST /my-index/_close 响应：
{ &#34;acknowledged&#34;: true, &#34;shards_acknowledged&#34;: true, &#34;indices&#34;: { &#34;my-index&#34;: { &#34;closed&#34;: true } } } 批量关闭：
POST /logs-2024-01,logs-2024-02/_close POST /logs-2024-*/_close 查询参数 #     参数 类型 默认值 说明     timeout Time 30s 操作超时时间   master_timeout Time 30s 连接主节点的超时时间   wait_for_active_shards String — 等待的活跃分片数（数字、all 或 index-setting）   expand_wildcards String open 通配符展开策略：open、closed、hidden、none、all   ignore_unavailable Boolean false 是否忽略不存在的索引   allow_no_indices Boolean true 通配符未匹配到索引时是否报错    打开索引 #  POST /my-index/_open 打开后索引会进入恢复流程（重建分片、分配副本），需要等待分片就绪后才能正常读写。"
---


# 开关索引与索引限制

## 关闭索引

关闭的索引**不消耗集群资源**（CPU、内存、文件句柄），但索引数据仍保留在磁盘上。关闭后索引不能读写，但可以随时重新打开。

典型场景：

- 历史索引暂时不需要查询，但不想删除
- 修改 Static 类型的索引设置（必须先关闭索引）
- 降低集群负载

```json
POST /my-index/_close
```

响应：

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "indices": {
    "my-index": {
      "closed": true
    }
  }
}
```

批量关闭：

```json
POST /logs-2024-01,logs-2024-02/_close
POST /logs-2024-*/_close
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `timeout` | Time | `30s` | 操作超时时间 |
| `master_timeout` | Time | `30s` | 连接主节点的超时时间 |
| `wait_for_active_shards` | String | — | 等待的活跃分片数（数字、`all` 或 `index-setting`） |
| `expand_wildcards` | String | `open` | 通配符展开策略：`open`、`closed`、`hidden`、`none`、`all` |
| `ignore_unavailable` | Boolean | `false` | 是否忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配到索引时是否报错 |

## 打开索引

```json
POST /my-index/_open
```

打开后索引会进入恢复流程（重建分片、分配副本），需要等待分片就绪后才能正常读写。

响应：

```json
{
  "acknowledged": true,
  "shards_acknowledged": true
}
```

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `timeout` | Time | `30s` | 操作超时时间 |
| `master_timeout` | Time | `30s` | 连接主节点的超时时间 |
| `wait_for_active_shards` | String | — | 等待的活跃分片数 |
| `expand_wildcards` | String | `closed` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 是否忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配到索引时是否报错 |

### 修改 Static 设置的典型流程

有些索引设置（如 `index.number_of_shards`、`index.codec`）是 **Static** 类型，只能在创建时或关闭状态下修改：

```json
// 1. 关闭索引
POST /my-index/_close

// 2. 修改 Static 设置
PUT /my-index/_settings
{
  "index.codec": "best_compression"
}

// 3. 重新打开
POST /my-index/_open
```

## 索引限制（Index Block）

索引限制可以精细控制索引的读写行为，比"关闭索引"更灵活。

### 添加索引限制

```json
PUT /my-index/_block/write
```

响应：

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "indices": [
    {
      "name": "my-index",
      "blocked": true
    }
  ]
}
```

### 可用的限制类型

| 限制类型 | 对应设置 | 说明 |
|--------|----------|------|
| `read_only` | `index.blocks.read_only` | 完全只读：禁止写入和元数据变更 |
| `read` | `index.blocks.read` | 禁止读操作 |
| `write` | `index.blocks.write` | 禁止写操作（但允许元数据变更） |
| `metadata` | `index.blocks.metadata` | 禁止元数据修改（settings、mappings 变更等） |
| `read_only_allow_delete` | `index.blocks.read_only_allow_delete` | 只读但允许删除索引（磁盘空间不足时系统自动设置） |

### 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `timeout` | Time | `30s` | 操作超时时间 |
| `master_timeout` | Time | `30s` | 连接主节点的超时时间 |
| `expand_wildcards` | String | `open` | 通配符展开策略 |
| `ignore_unavailable` | Boolean | `false` | 是否忽略不存在的索引 |
| `allow_no_indices` | Boolean | `true` | 通配符未匹配到索引时是否报错 |

### 通过 Settings API 设置/移除限制

也可以直接通过 `_settings` API 管理限制：

```json
// 设为只读
PUT /my-index/_settings
{
  "index.blocks.write": true
}

// 移除只读
PUT /my-index/_settings
{
  "index.blocks.write": false
}
```

## 使用建议

| 场景 | 推荐方式 |
|------|----------|
| 索引归档，长期不查询 | `_close` 关闭索引 |
| 索引仍需查询但禁止写入 | `_block/write` 或 `index.blocks.write: true` |
| 修改 Static 设置 | 先 `_close`，改完再 `_open` |
| 磁盘空间紧张时自动保护 | 系统自动设置 `read_only_allow_delete`，释放空间后手动移除 |

## 下一步

- [索引设置](./index-settings.md)：了解 Static 与 Dynamic 设置的完整列表
- [克隆/缩小/拆分](./clone-shrink-split.md)：调整索引分片结构
- [索引管理](./index-management.md)：索引的创建与删除


---
title: "索引滚动（Rollover）"
date: 0001-01-01
description: "当索引达到年龄、文档数或大小阈值时，自动滚动到新索引。"
summary: "索引滚动（Rollover） #  Rollover API 用于在满足特定条件时，将一个别名或数据流（Data Stream）滚动到一个新的索引。这是管理时间序列数据的核心操作——让你的索引保持合理大小，避免单个索引无限膨胀。
基本用法 #  前置条件 #  Rollover 只能在以下两种目标上执行：
 写入别名（Write Alias）：别名必须标记了 is_write_index: true 数据流（Data Stream）：天然支持 Rollover  对别名执行 Rollover #  先创建索引和写入别名：
PUT /logs-000001 { &#34;aliases&#34;: { &#34;logs-write&#34;: { &#34;is_write_index&#34;: true } } } 当条件满足时执行 Rollover：
POST /logs-write/_rollover { &#34;conditions&#34;: { &#34;max_age&#34;: &#34;7d&#34;, &#34;max_docs&#34;: 10000000, &#34;max_size&#34;: &#34;50gb&#34;, &#34;max_primary_shard_size&#34;: &#34;25gb&#34; } } 响应：
{ &#34;acknowledged&#34;: true, &#34;shards_acknowledged&#34;: true, &#34;old_index&#34;: &#34;logs-000001&#34;, &#34;new_index&#34;: &#34;logs-000002&#34;, &#34;rolled_over&#34;: true, &#34;dry_run&#34;: false, &#34;conditions&#34;: { &#34;[max_age: 7d]&#34;: false, &#34;[max_docs: 10000000]&#34;: true, &#34;[max_size: 50gb]&#34;: false, &#34;[max_primary_shard_size: 25gb]&#34;: false } }  任一条件满足即触发滚动。响应中会列出每个条件的评估结果。"
---


# 索引滚动（Rollover）

Rollover API 用于在满足特定条件时，将一个别名或数据流（Data Stream）滚动到一个新的索引。这是管理时间序列数据的核心操作——让你的索引保持合理大小，避免单个索引无限膨胀。

## 基本用法

### 前置条件

Rollover 只能在以下两种目标上执行：

1. **写入别名（Write Alias）**：别名必须标记了 `is_write_index: true`
2. **数据流（Data Stream）**：天然支持 Rollover

### 对别名执行 Rollover

先创建索引和写入别名：

```json
PUT /logs-000001
{
  "aliases": {
    "logs-write": {
      "is_write_index": true
    }
  }
}
```

当条件满足时执行 Rollover：

```json
POST /logs-write/_rollover
{
  "conditions": {
    "max_age": "7d",
    "max_docs": 10000000,
    "max_size": "50gb",
    "max_primary_shard_size": "25gb"
  }
}
```

响应：

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "old_index": "logs-000001",
  "new_index": "logs-000002",
  "rolled_over": true,
  "dry_run": false,
  "conditions": {
    "[max_age: 7d]": false,
    "[max_docs: 10000000]": true,
    "[max_size: 50gb]": false,
    "[max_primary_shard_size: 25gb]": false
  }
}
```

> **任一条件满足**即触发滚动。响应中会列出每个条件的评估结果。

### 对数据流执行 Rollover

```json
POST /logs-nginx/_rollover
{
  "conditions": {
    "max_age": "1d",
    "max_docs": 5000000
  }
}
```

### 指定新索引名

默认情况下，新索引名会在旧索引名基础上递增数字后缀（如 `logs-000001` → `logs-000002`）。也可以指定自定义名称：

```json
POST /logs-write/_rollover/logs-2024-07-01
{
  "conditions": {
    "max_age": "7d"
  }
}
```

## 滚动条件

| 条件 | 类型 | 说明 |
|------|------|------|
| `max_age` | Time | 索引创建后经过的时间（如 `"7d"`、`"30d"`） |
| `max_docs` | Long | 索引中的文档总数 |
| `max_size` | Size | 索引的总大小（如 `"50gb"`） |
| `max_primary_shard_size` | Size | 最大主分片的大小 |

> **`max_primary_shard_size`** 是控制分片大小最实用的条件。推荐单个分片大小维持在 10–50GB 之间。

## Dry Run：模拟执行

使用 `dry_run` 参数可以预览 Rollover 结果而不实际执行：

```json
POST /logs-write/_rollover?dry_run
{
  "conditions": {
    "max_docs": 1000000
  }
}
```

## 为新索引指定设置

可以在请求体中为新索引指定设置、映射和别名：

```json
POST /logs-write/_rollover
{
  "conditions": {
    "max_age": "7d"
  },
  "settings": {
    "index.number_of_shards": 3,
    "index.number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "message": { "type": "text" }
    }
  },
  "aliases": {
    "logs-read": {}
  }
}
```

> 通常建议使用[索引模板](./index-templates.md)来统一管理新索引的设置和映射，而不是在每次 Rollover 时手动指定。

## 查询参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `dry_run` | Boolean | `false` | 模拟执行，不实际创建新索引 |
| `timeout` | Time | `30s` | 操作超时时间 |
| `master_timeout` | Time | `30s` | 连接主节点的超时时间 |
| `wait_for_active_shards` | String | — | 等待的活跃分片数 |

## 索引命名约定

要让自动递增的数字后缀生效，索引名必须以 `-` 和数字结尾，且数字需要足够的前导零：

| 当前索引 | Rollover 后 |
|----------|-------------|
| `logs-000001` | `logs-000002` |
| `logs-2024-01-000001` | `logs-2024-01-000002` |
| `my-index-3` | `my-index-000004` |

> 建议统一使用 6 位数字后缀（如 `-000001`），为未来的滚动留出足够空间。

## 与 ILM 配合使用

在生产环境中，通常不手动调用 Rollover API，而是通过 [索引生命周期管理（ILM）]({{< relref "/docs/features/data-retention/ilm" >}}) 策略自动触发 Rollover。ILM 会定期检查条件，满足时自动执行滚动。

典型的 ILM + Rollover 流程：

1. 定义 ILM 策略，在 hot 阶段设置 Rollover 条件
2. 创建索引模板，关联 ILM 策略
3. 创建初始索引和写入别名
4. ILM 自动管理后续的滚动、迁移和删除

## 下一步

- [索引模板](./index-templates.md)：为 Rollover 创建的新索引预配设置
- [别名（Aliases）](./aliases.md)：写入别名的管理
- [数据流](./data-streams.md)：天然支持 Rollover 的时序数据管理
- [索引生命周期管理]({{< relref "/docs/features/data-retention/ilm" >}})：自动化 Rollover 策略


---
title: "克隆、缩小与拆分索引"
date: 0001-01-01
description: "使用 Clone、Shrink、Split API 调整索引的分片结构。"
summary: "克隆、缩小与拆分索引 #  这三个操作都是&quot;把一个现有索引复制到一个新索引&quot;，区别在于目标索引的主分片数：
   操作 API 目标主分片数 典型场景     Clone _clone 与源索引相同 复制索引用于测试/实验   Shrink _shrink 源分片数的因子（如 6→3、6→2、6→1） 合并小分片、降低开销   Split _split 源分片数的倍数（如 2→4、2→6） 扩展分片以提高写入并发     前置条件：源索引必须是只读状态。所有操作都会创建一个全新的索引，源索引保持不变。
 公共前置步骤 #  在执行 Clone / Shrink / Split 之前，源索引必须标记为只读：
PUT /source-index/_settings { &#34;index.blocks.write&#34;: true } 对于 Shrink 操作，还需要将所有分片的副本迁移到同一个节点：
PUT /source-index/_settings { &#34;index.routing.allocation.require._name&#34;: &#34;node-1&#34;, &#34;index.blocks.write&#34;: true } 等待所有分片完成迁移：
GET _cat/shards/source-index?v 确认所有分片都在目标节点上后再执行 Shrink。"
---


# 克隆、缩小与拆分索引

这三个操作都是"把一个现有索引复制到一个新索引"，区别在于目标索引的主分片数：

| 操作 | API | 目标主分片数 | 典型场景 |
|------|-----|-------------|----------|
| **Clone** | `_clone` | 与源索引相同 | 复制索引用于测试/实验 |
| **Shrink** | `_shrink` | 源分片数的因子（如 6→3、6→2、6→1） | 合并小分片、降低开销 |
| **Split** | `_split` | 源分片数的倍数（如 2→4、2→6） | 扩展分片以提高写入并发 |

> **前置条件**：源索引必须是**只读**状态。所有操作都会创建一个**全新的索引**，源索引保持不变。

## 公共前置步骤

在执行 Clone / Shrink / Split 之前，源索引必须标记为只读：

```json
PUT /source-index/_settings
{
  "index.blocks.write": true
}
```

对于 Shrink 操作，还需要将所有分片的副本迁移到同一个节点：

```json
PUT /source-index/_settings
{
  "index.routing.allocation.require._name": "node-1",
  "index.blocks.write": true
}
```

等待所有分片完成迁移：

```json
GET _cat/shards/source-index?v
```

确认所有分片都在目标节点上后再执行 Shrink。

## Clone：克隆索引

将索引完整克隆为一个新索引（相同的分片数、设置和数据）。

```json
POST /source-index/_clone/target-index
```

可以在请求体中为目标索引指定不同的设置：

```json
POST /source-index/_clone/target-index
{
  "settings": {
    "index.number_of_replicas": 2
  },
  "aliases": {
    "my-alias": {}
  }
}
```

克隆完成后，记得移除源索引的写入块：

```json
PUT /source-index/_settings
{
  "index.blocks.write": false
}
```

## Shrink：缩小索引

将索引缩小为更少的主分片。目标分片数必须是源分片数的**因子**。

例如，一个 12 分片的索引可以缩小为 6、4、3、2 或 1 个分片。

```json
POST /logs-000001/_shrink/logs-000001-shrunk
{
  "settings": {
    "index.number_of_shards": 1,
    "index.number_of_replicas": 1,
    "index.routing.allocation.require._name": null,
    "index.blocks.write": null
  }
}
```

> **注意**：目标索引的设置中通常需要清除 `routing.allocation.require` 和 `blocks.write`，否则新索引会继承这些限制。

### Shrink 的工作原理

Shrink 使用硬链接（hard link）而不是复制数据，所以速度很快且几乎不消耗额外磁盘空间。具体过程：

1. 创建目标索引，主分片数为指定值
2. 将源索引的段（segment）通过硬链接映射到目标索引
3. 对目标索引进行恢复（如同刚打开的索引）

## Split：拆分索引

将索引拆分为更多的主分片。目标分片数必须是源分片数的**倍数**。

例如，一个 2 分片的索引可以拆分为 4、6、8 等。

```json
POST /my-index/_split/my-index-split
{
  "settings": {
    "index.number_of_shards": 4
  }
}
```

### Split 的前置条件

源索引创建时，`index.number_of_routing_shards` 决定了拆分的上限。例如：

```json
PUT /my-index
{
  "settings": {
    "index.number_of_shards": 2,
    "index.number_of_routing_shards": 8
  }
}
```

此索引最多可以拆分到 8 个分片（2→4→8）。

## 公共查询参数

三个 API 共享相同的查询参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `timeout` | Time | `30s` | 操作超时时间 |
| `master_timeout` | Time | `30s` | 连接主节点的超时时间 |
| `wait_for_active_shards` | String | — | 等待的活跃分片数（数字、`all` 或 `index-setting`） |

## 请求体

三个 API 的请求体结构相同：

| 字段 | 类型 | 说明 |
|------|------|------|
| `settings` | Object | 目标索引的设置（会覆盖从源索引继承的值） |
| `aliases` | Object | 目标索引的别名定义 |

## 操作完成后

完成后通常需要：

1. 移除源索引的写入块（如果还需要使用源索引）
2. 验证目标索引的数据完整性
3. 如果不再需要源索引，可以删除或关闭它
4. 如果使用了别名，更新别名指向

```json
// 验证文档数
GET /target-index/_count

// 通过别名切换
POST _aliases
{
  "actions": [
    { "remove": { "index": "source-index", "alias": "my-alias" } },
    { "add": { "index": "target-index", "alias": "my-alias" } }
  ]
}
```

## 下一步

- [索引管理](./index-management.md)：创建与删除索引
- [开关索引与索引限制](./open-close-index.md)：设置只读限制
- [别名（Aliases）](./aliases.md)：零停机切换索引
- [索引设置](./index-settings.md)：分片与副本配置


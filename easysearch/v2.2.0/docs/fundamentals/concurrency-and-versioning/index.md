---
title: "并发控制与版本"
date: 0001-01-01
description: "乐观并发控制、版本控制与安全更新模式。"
summary: "并发控制与版本 #  当有多个客户端同时写入同一份文档时，如果不做任何并发控制，很容易出现&quot;旧值覆盖新值&quot;的问题。本页介绍如何用版本与乐观锁避免这类隐性数据错误。
为什么需要并发控制？ #  考虑这样一个场景：
 客户端 A 读取文档，准备把字段 count 从 1 改到 2 客户端 B 也读取了同一文档，把 count 从 1 改到 3  如果没有并发控制：
 A 先写入，文档变成 count=2 B 后写入，文档被覆盖为 count=3（A 的更新&quot;丢了&quot;）  很多时候这种问题不会立刻暴露，但会在数据对账或业务逻辑中造成难以解释的异常。
乐观并发控制（Optimistic Concurrency Control） #  Easysearch 使用 _seq_no（序列号） 和 _primary_term（主分片任期） 实现乐观并发控制：
 _seq_no：每次对分片的写操作递增，全局唯一标识该分片上的操作顺序 _primary_term：当主分片发生切换（故障转移）时递增，用于区分不同&quot;任期&quot;的写入  每次读取文档时，响应中都会包含这两个值：
GET /products/_doc/1 // 响应 { &#34;_index&#34;: &#34;products&#34;, &#34;_type&#34;: &#34;_doc&#34;, &#34;_id&#34;: &#34;1&#34;, &#34;_version&#34;: 3, &#34;_seq_no&#34;: 5, &#34;_primary_term&#34;: 1, &#34;found&#34;: true, &#34;_source&#34;: { &#34;name&#34;: &#34;iPhone&#34;, &#34;count&#34;: 10, &#34;price&#34;: 5999 } } 安全更新：if_seq_no + if_primary_term #  使用 if_seq_no 和 if_primary_term 参数，只有当前版本与预期一致时才允许更新："
---


# 并发控制与版本

当有多个客户端同时写入同一份文档时，如果不做任何并发控制，很容易出现"旧值覆盖新值"的问题。本页介绍如何用版本与乐观锁避免这类隐性数据错误。

## 为什么需要并发控制？

考虑这样一个场景：

- 客户端 A 读取文档，准备把字段 `count` 从 1 改到 2
- 客户端 B 也读取了同一文档，把 `count` 从 1 改到 3

如果没有并发控制：

- A 先写入，文档变成 `count=2`
- B 后写入，文档被覆盖为 `count=3`（A 的更新"丢了"）

很多时候这种问题不会立刻暴露，但会在数据对账或业务逻辑中造成难以解释的异常。

## 乐观并发控制（Optimistic Concurrency Control）

Easysearch 使用 **`_seq_no`（序列号）** 和 **`_primary_term`（主分片任期）** 实现乐观并发控制：

- `_seq_no`：每次对分片的写操作递增，全局唯一标识该分片上的操作顺序
- `_primary_term`：当主分片发生切换（故障转移）时递增，用于区分不同"任期"的写入

每次读取文档时，响应中都会包含这两个值：

```json
GET /products/_doc/1

// 响应
{
  "_index": "products",
  "_type": "_doc",
  "_id": "1",
  "_version": 3,
  "_seq_no": 5,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "iPhone",
    "count": 10,
    "price": 5999
  }
}
```

## 安全更新：if_seq_no + if_primary_term

使用 `if_seq_no` 和 `if_primary_term` 参数，只有当前版本与预期一致时才允许更新：

```json
// 基于读取到的 _seq_no=5, _primary_term=1 进行更新
PUT /products/_doc/1?if_seq_no=5&if_primary_term=1
{
  "name": "iPhone",
  "count": 9,
  "price": 5999
}
```

- **版本匹配**：更新成功，`_seq_no` 递增
- **版本不匹配**（中途有人修改过）：返回 `409 Conflict`

```json
// 冲突响应
{
  "error": {
    "type": "version_conflict_engine_exception",
    "reason": "[1]: version conflict, required seqNo [5], primary term [1]. current document has seqNo [6] and primary term [1]"
  },
  "status": 409
}
```

`_update` API 同样支持这两个参数：

```json
POST /products/_doc/1/_update?if_seq_no=5&if_primary_term=1
{
  "doc": {
    "count": 9
  }
}
```

## 外部版本（External Versioning）

当数据从外部系统（关系数据库、消息队列）同步到 Easysearch 时，可以使用外部系统的版本号来控制写入顺序：

```json
// 使用外部版本号
PUT /products/_doc/1?version=100&version_type=external
{
  "name": "iPhone",
  "count": 10,
  "price": 5999
}
```

`version_type` 的取值：

| 类型 | 行为 |
|------|------|
| `internal`（默认） | 使用 Easysearch 内部版本号，要求精确匹配 |
| `external` / `external_gte` | 使用外部版本号；只要新版本 > 当前版本就允许写入（`external_gte` 允许 ≥） |

这在数据库→Easysearch 同步场景中非常实用：数据库的自增主键或时间戳可以直接作为版本号，保证更新顺序。

## retry_on_conflict：自动重试

对于"计数器类、顺序不敏感"的 update，可以通过 `retry_on_conflict` 自动重试冲突：

```json
POST /products/_doc/1/_update?retry_on_conflict=3
{
  "script": "ctx._source.count += 1"
}
```

遇到版本冲突时，Easysearch 会自动重新读取最新文档并重试 update，最多重试指定次数。

> **适用场景**：计数器累加、浏览量统计等"最终值正确即可"的场景。对于顺序敏感的业务（如库存扣减），不要依赖自动重试——应在客户端进行冲突处理。

## 幂等性设计

有了并发控制之后，重试逻辑需要考虑幂等性：

- **版本冲突类错误**：盲目重试没有意义——需要回读最新文档，再基于新状态重新计算更新
- **临时性错误**（网络抖动、资源不足）：可以在保证幂等的前提下做重试

实现幂等的常用手段：

- 使用稳定的 `_id`（按业务唯一键），相同 `_id` 的 `index` 操作天然幂等（覆盖写）
- 使用 `op_type=create`，保证同一 `_id` 只创建一次
- 在文档中显式记录操作版本/时间戳，在更新逻辑中进行检查

## 与业务语义的结合

并发控制策略要与业务语义配合设计：

| 场景 | 推荐策略 |
|------|----------|
| 累计类字段（总访问量） | `retry_on_conflict` + 脚本自增，关注整体趋势即可 |
| 库存 / 余额 / 配额 | `if_seq_no` + `if_primary_term` 严格乐观锁，客户端处理冲突 |
| 状态类字段（订单状态） | 乐观锁 + 状态机收敛到单一负责方 |
| 外部系统同步 | `version_type=external`，以外部系统版本号为准 |
| 日志 / 事件 | `op_type=create` + 稳定 `_id`，保证不重复写入 |

## 小结

- `_seq_no` + `_primary_term` 是 Easysearch 中乐观并发控制的基础，通过 `if_seq_no` 和 `if_primary_term` 参数实现条件更新
- `version_type=external` 适用于从外部系统同步数据的场景
- `retry_on_conflict` 适用于顺序不敏感的累加类操作
- 设计文档结构与更新流程时，要预先想清楚并发模型与幂等策略

下一步可以继续阅读：

- [_source 与存储字段]({{< relref "./source-and-stored-fields.md" >}})
- [索引设置]({{< relref "/docs/operations/data-management/index-settings.md" >}})
- [文档设计]({{< relref "/docs/best-practices/data-modeling/document-design.md" >}})




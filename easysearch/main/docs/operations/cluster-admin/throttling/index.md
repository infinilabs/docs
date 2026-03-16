---
title: "写入限流"
date: 0001-01-01
description: "节点级、索引级、分片级三层限速控制，保障写入不影响查询性能。"
summary: "写入限流 #  Easysearch 从 1.8.0 版本开始引入写入限流功能，支持在节点、索引和分片三个层级对写入速度进行精细化控制。在需要向正在执行查询的集群导入数据时，限流功能可以有效避免写入压力影响查询响应时间。
 三级限流架构 #     级别 粒度 适用场景     节点级 整个节点的写入总量 保护节点整体性能，不区分索引   索引级 单个索引的写入速度 限制特定索引，不影响其他索引   分片级 每个分片的写入速度 适合高低配主机混搭的集群     节点级和分片级限流可以同时启用，互不冲突。限流功能不会限制系统索引流量，只针对业务索引。
 限流参数参考 #  以下参数均支持动态设置，无需重启集群。
   参数 类型 说明 默认值     cluster.throttle.node.write boolean 是否启用节点级别限流 false   cluster.throttle.node.write.max_requests int 限定时间范围内单个节点允许的最大写入请求次数 0   cluster.throttle.node.write.max_bytes 字符串 限定时间范围内单个节点允许的最大写入请求字节数（kb, mb, gb 等） 0mb   cluster."
---


# 写入限流

Easysearch 从 1.8.0 版本开始引入写入限流功能，支持在节点、索引和分片三个层级对写入速度进行精细化控制。在需要向正在执行查询的集群导入数据时，限流功能可以有效避免写入压力影响查询响应时间。

---

## 三级限流架构

| 级别 | 粒度 | 适用场景 |
|------|------|----------|
| **节点级** | 整个节点的写入总量 | 保护节点整体性能，不区分索引 |
| **索引级** | 单个索引的写入速度 | 限制特定索引，不影响其他索引 |
| **分片级** | 每个分片的写入速度 | 适合高低配主机混搭的集群 |

> 节点级和分片级限流可以同时启用，互不冲突。限流功能不会限制系统索引流量，只针对业务索引。

## 限流参数参考

以下参数均支持动态设置，无需重启集群。

| 参数 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| cluster.throttle.node.write | boolean | 是否启用节点级别限流                             | false |
| cluster.throttle.node.write.max_requests | int     | 限定时间范围内单个节点允许的最大写入请求次数                 | 0   |
| cluster.throttle.node.write.max_bytes | 字符串     | 限定时间范围内单个节点允许的最大写入请求字节数（kb, mb, gb 等）  | 0mb |
| cluster.throttle.node.write.action | 字符串     | 	触发限速之后的处理动作，分为 retry 和 drop 两种，默认为 drop | drop |
| cluster.throttle.node.write.interval | int     | 节点级别评估限速的单位时间间隔，默认为 1s                 | 1   |
| cluster.throttle.shard.write | boolean | 是否启用分片级别限流                             | false |
| cluster.throttle.shard.write.max_requests | int     | 限定时间范围内单个分片允许的最大写入请求次数                 | 0   |
| cluster.throttle.shard.write.max_bytes | 字符串     | 限定时间范围内单个分片允许的最大写入请求字节数（kb, mb, gb 等）  | 0mb |
| cluster.throttle.shard.write.action | 字符串     | 触发限速之后的处理动作，分为 retry 和 drop 两种，默认为 drop | drop |
| cluster.throttle.shard.write.interval | int     | 分片级别评估限速的单位时间间隔，默认为 1s                 | 1   |
| index.throttle.write.enable | boolean | 是否针对当前索引启用写入限流（索引 Settings 配置）         | false |
| index.throttle.write.max_requests | int     | 限定时间范围内当前索引允许的最大写入请求次数（索引 Settings 配置）               | 0   |
| index.throttle.write.max_bytes | 字符串     | 限定时间范围内当前索引允许的最大写入请求字节数（kb, mb, gb 等）（索引 Settings 配置） | 0mb |
| index.throttle.write.action | 字符串     | 触发限速之后的处理动作，分为 retry 和 drop 两种，默认为 drop（索引 Settings 配置） | drop |
| index.throttle.write.interval | int     | 当前索引评估限速的单位时间间隔，默认为 1s（索引 Settings 配置）               | 1   |

## 使用示例

### 节点级别限流
```
PUT _cluster/settings
{
  "transient": {
    "cluster.throttle.node.write": true,
    "cluster.throttle.node.write.max_bytes": "50MB",
    "cluster.throttle.node.write.max_requests": 1000000,
    "cluster.throttle.node.write.action": "retry"
  }
}

```
以上配置表示开启节点限流功能，限定时间范围内单个节点允许最大写入50MB的数据，并且写入条数限制在100万，超过设定的阈值后会持续重试1秒钟，实际流量计算会稍有偏差。

### 分片级别限流
```
PUT _cluster/settings
{
  "transient": {
    "cluster.throttle.shard.write": true,
    "cluster.throttle.shard.write.max_bytes": "50MB",
    "cluster.throttle.shard.write.max_requests": 1000000,
    "cluster.throttle.shard.write.action": "drop"
  }
}

```
以上配置表示开启分片限流功能，限定时间范围内单个分片允许最大写入50MB的数据，并且写入条数限制在100万，超过设定的阈值后会立即拒绝写入，返回 rejected execution， 实际流量计算会稍有偏差。

### 索引级别限流
有时，我们需要只针对个别索引进行写入限流，又不想影响其他索引的写入速度，可以在创建索引时在 Settings 里指定相应的限流配置项：

```
PUT test_0
{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 3,
    "index.throttle.write.max_requests": 6000,
    "index.throttle.write.action": "retry",
    "index.throttle.write.enable": true
  }
}
```

## action 参数说明

| 值 | 行为 |
|-----|------|
| `retry` | 超限的请求进行短暂重试，在间隔时间后重新评估，对客户端更友好 |
| `drop` | 直接丢弃超限的请求（返回 rejected execution），适合允许数据丢失的场景 |

---

## 注意事项

- 节点级别限流针对所有 DataNode
- 分片级别限流只计算从协调节点分发到数据节点主分片的 bulk 请求
- 节点级别和分片级别限流不冲突，可以同时启用
- 限流功能不会限制系统索引（`.` 开头）的流量，只针对业务索引
- 所有限流参数均支持动态设置，无需重启集群

---

## 相关文档

- [文档操作]({{< relref "/docs/features/document-operations/_index.md" >}})
- [容量规划]({{< relref "/docs/best-practices/index-design.md" >}})


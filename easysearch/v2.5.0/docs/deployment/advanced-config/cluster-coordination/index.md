---
title: "集群协调调优"
date: 0001-01-01
summary: "集群协调调优 #  本文介绍 Easysearch 集群协调层的高级调优参数，包括选举、故障检测和集群状态发布。这些参数通常不需要修改，仅在大规模集群或网络条件不佳时才需要调整。
 前提：请先阅读 集群发现 了解基础配置。
  选举调优 #  选举机制控制集群在主节点失联后如何选出新的主节点。
cluster.election.initial_timeout #     项目 说明     参数 cluster.election.initial_timeout   默认值 100ms   属性 静态   说明 节点首次发起选举前的随机等待上界。初始超时引入随机性，避免多个节点同时发起选举导致票数分裂    cluster.election.back_off_time #     项目 说明     参数 cluster.election.back_off_time   默认值 100ms   属性 静态   说明 每次选举失败后增加的退避时间上界。防止选举风暴    cluster."
---


# 集群协调调优

本文介绍 Easysearch 集群协调层的高级调优参数，包括选举、故障检测和集群状态发布。这些参数通常不需要修改，仅在大规模集群或网络条件不佳时才需要调整。

> **前提**：请先阅读 [集群发现]({{< relref "../config/node-settings/discovery.md" >}}) 了解基础配置。

---

## 选举调优

选举机制控制集群在主节点失联后如何选出新的主节点。

### cluster.election.initial_timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.election.initial_timeout` |
| **默认值** | `100ms` |
| **属性** | 静态 |
| **说明** | 节点首次发起选举前的随机等待上界。初始超时引入随机性，避免多个节点同时发起选举导致票数分裂 |

### cluster.election.back_off_time

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.election.back_off_time` |
| **默认值** | `100ms` |
| **属性** | 静态 |
| **说明** | 每次选举失败后增加的退避时间上界。防止选举风暴 |

### cluster.election.max_timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.election.max_timeout` |
| **默认值** | `10s` |
| **属性** | 静态 |
| **说明** | 选举超时的最大值。选举超时从 `initial_timeout` 开始，每次失败按 `back_off_time` 递增，但不超过此值 |

### cluster.election.duration

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.election.duration` |
| **默认值** | `500ms` |
| **属性** | 静态 |
| **说明** | 单次选举允许的最长时间。超过此时间仍未完成选举则重新开始 |

---

## 故障检测

故障检测机制用于识别已下线的节点，分为 **leader 检测**（非主节点检测主节点是否存活）和 **follower 检测**（主节点检测其他节点是否存活）。

### cluster.fault_detection.leader_check.interval

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.fault_detection.leader_check.interval` |
| **默认值** | `1s` |
| **属性** | 静态 |
| **说明** | 非主节点向主节点发送心跳的间隔 |

### cluster.fault_detection.leader_check.timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.fault_detection.leader_check.timeout` |
| **默认值** | `10s` |
| **属性** | 静态 |
| **说明** | 心跳请求超时时间。超时则视为一次检测失败 |

### cluster.fault_detection.leader_check.retry_count

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.fault_detection.leader_check.retry_count` |
| **默认值** | `3` |
| **最小值** | `1` |
| **属性** | 静态 |
| **说明** | 连续检测失败多少次后认为主节点已下线，触发重新选举 |

### cluster.fault_detection.follower_check.interval

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.fault_detection.follower_check.interval` |
| **默认值** | `1s` |
| **属性** | 静态 |
| **说明** | 主节点向其他节点发送心跳的间隔 |

### cluster.fault_detection.follower_check.timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.fault_detection.follower_check.timeout` |
| **默认值** | `10s` |
| **属性** | 静态 |
| **说明** | 主节点向其他节点发送心跳的超时时间 |

### cluster.fault_detection.follower_check.retry_count

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.fault_detection.follower_check.retry_count` |
| **默认值** | `3` |
| **最小值** | `1` |
| **属性** | 静态 |
| **说明** | 主节点连续检测失败多少次后将该节点从集群中移除 |

---

## 集群状态发布

集群状态（Cluster State）包含索引映射、分片分配等元数据，主节点在每次变更时将新状态发布到所有节点。

### cluster.publish.timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.publish.timeout` |
| **默认值** | `30s` |
| **属性** | 静态 |
| **说明** | 集群状态发布的完整超时时间（包含提交阶段）。超时则视为发布失败 |

### cluster.publish.info_timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.publish.info_timeout` |
| **默认值** | `10s` |
| **属性** | 静态 |
| **说明** | 发布信息阶段的超时时间。超时后会记录详细日志，但不会导致发布失败 |

---

## 其他协调参数

### cluster.join.timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.join.timeout` |
| **默认值** | `60s` |
| **属性** | 静态 |
| **说明** | 节点加入集群的超时时间。超时后节点将重试加入 |

### cluster.follower_lag.timeout

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.follower_lag.timeout` |
| **默认值** | `90s` |
| **属性** | 静态 |
| **说明** | 跟随节点滞后超时。如果跟随节点在此时间内未确认集群状态更新，主节点可能将其视为滞后节点 |

### cluster.no_master_block

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.no_master_block` |
| **默认值** | `write` |
| **可选值** | `all`、`write` |
| **属性** | 动态 |
| **说明** | 集群没有主节点时的行为。`write` = 拒绝写入但允许读取；`all` = 拒绝所有操作 |

---

## 调优建议

### 大规模集群（>50 节点）

大集群中状态发布和故障检测的负担更重：

```yaml
# 增加发布超时
cluster.publish.timeout: 60s

# 增加心跳容忍度，避免误判
cluster.fault_detection.leader_check.retry_count: 5
cluster.fault_detection.follower_check.retry_count: 5
```

### 网络不稳定环境

跨机房或网络延迟较高时：

```yaml
# 增加检测超时
cluster.fault_detection.leader_check.timeout: 30s
cluster.fault_detection.follower_check.timeout: 30s

# 增加选举超时
cluster.election.max_timeout: 30s

# 增加加入超时
cluster.join.timeout: 120s
```

### 快速故障转移

对可用性要求极高的场景，缩短检测间隔（会增加网络开销）：

```yaml
cluster.fault_detection.leader_check.interval: 500ms
cluster.fault_detection.leader_check.timeout: 5s
cluster.fault_detection.leader_check.retry_count: 2

cluster.fault_detection.follower_check.interval: 500ms
cluster.fault_detection.follower_check.timeout: 5s
cluster.fault_detection.follower_check.retry_count: 2
```

> ⚠️ **注意**：过于激进的故障检测设置可能导致 GC 停顿被误判为节点下线。请确保 JVM 配置合理。

---

## 延伸阅读

- [集群发现配置]({{< relref "../config/node-settings/discovery.md" >}}) - 基础发现参数
- [集群节点配置]({{< relref "../config/node-settings/cluster-node.md" >}}) - 节点角色配置
- [网络配置]({{< relref "../config/node-settings/network.md" >}}) - 传输层网络配置


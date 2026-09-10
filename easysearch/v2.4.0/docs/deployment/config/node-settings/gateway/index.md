---
title: "网关与恢复"
date: 0001-01-01
summary: "网关与恢复配置 #  本页介绍 easysearch.yml 中与集群完全重启后分片恢复行为相关的配置项。这些都是静态设置，修改后需要重启节点生效。
 分片恢复的限流参数（如 indices.recovery.max_bytes_per_sec）属于动态配置，通过 集群配置 API 修改。
  为什么需要网关恢复设置？ #  当整个集群重启时，各节点可能以不同的速度启动。如果集群在只有部分节点加入时就立即开始分片恢复，会导致：
 不必要的数据搬运：节点 A 先启动，集群把本该在节点 B 上的分片副本重新分配给 A；等 B 启动后又要搬回去。 大量网络 I/O：分片数据在节点间来回传输。 恢复时间变长：所有数据都要重新复制一遍。  网关设置让集群等到足够数量的节点加入后再开始恢复，避免上述问题。
 gateway.recover_after_nodes #  gateway.recover_after_nodes: 2    项目 说明     参数 gateway.recover_after_nodes   默认值 -1 (禁用)   属性 静态   说明 集群中至少有多少个节点加入后才开始恢复分片。已弃用：建议改用 recover_after_data_nodes 和 recover_after_master_nodes 以获得更精确的控制    推荐值 #   三节点集群：设为 2 一般规则：设为集群总节点数的大多数（$\lceil N/2 \rceil + 1$ 或更大）  gateway."
---


# 网关与恢复配置

本页介绍 `easysearch.yml` 中与集群完全重启后分片恢复行为相关的配置项。这些都是**静态设置**，修改后需要重启节点生效。

> 分片恢复的**限流参数**（如 `indices.recovery.max_bytes_per_sec`）属于动态配置，通过 [集群配置 API]({{< relref "../configuration.md" >}}) 修改。

---

## 为什么需要网关恢复设置？

当整个集群重启时，各节点可能以不同的速度启动。如果集群在只有部分节点加入时就立即开始分片恢复，会导致：

1. **不必要的数据搬运**：节点 A 先启动，集群把本该在节点 B 上的分片副本重新分配给 A；等 B 启动后又要搬回去。
2. **大量网络 I/O**：分片数据在节点间来回传输。
3. **恢复时间变长**：所有数据都要重新复制一遍。

网关设置让集群等到足够数量的节点加入后再开始恢复，避免上述问题。

---

## gateway.recover_after_nodes

```yaml
gateway.recover_after_nodes: 2
```

| 项目 | 说明 |
|------|------|
| **参数** | `gateway.recover_after_nodes` |
| **默认值** | `-1` (禁用) |
| **属性** | 静态 |
| **说明** | 集群中至少有多少个节点加入后才开始恢复分片。**已弃用**：建议改用 `recover_after_data_nodes` 和 `recover_after_master_nodes` 以获得更精确的控制 |

### 推荐值

- **三节点集群**：设为 `2`
- 一般规则：设为集群总节点数的大多数（$\lceil N/2 \rceil + 1$ 或更大）

## gateway.expected_nodes

```yaml
gateway.expected_nodes: 3
```

| 项目 | 说明 |
|------|------|
| **参数** | `gateway.expected_nodes` |
| **默认值** | `-1` (禁用) |
| **属性** | 静态 |
| **说明** | 集群预期的总节点数。当加入的节点数达到此值时，立即开始恢复（不等待 `recover_after_time`）。**已弃用**：建议改用 `expected_data_nodes` |

## gateway.recover_after_time

```yaml
# 生产环境建议显式设置
gateway.recover_after_time: 5m
```

| 项目 | 说明 |
|------|------|
| **参数** | `gateway.recover_after_time` |
| **默认值** | `0ms` (无延迟) |
| **属性** | 静态 |
| **说明** | 达到 `recover_after_nodes` 后，再等待多长时间才开始恢复。给剩余节点留出加入集群的时间窗口。如果在此期间达到 `expected_nodes`，则提前开始恢复。**生产环境建议设为 `5m`** |

---

## 数据节点专用参数

如果集群中有角色分离（专用 master + 专用 data），可以单独设置数据节点的恢复条件：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `gateway.expected_data_nodes` | `-1` | 预期的数据节点数。达到后立即开始恢复 |
| `gateway.recover_after_data_nodes` | `-1` | 至少有多少数据节点加入后才开始恢复 |
| `gateway.expected_master_nodes` | `-1` | 预期的 master 候选节点数（已弃用） |
| `gateway.recover_after_master_nodes` | `0` | 至少有多少 master 候选节点加入后才开始恢复（已弃用），默认跟随 `discovery.zen.minimum_master_nodes` |

---

## 恢复流程示意

```
集群全量重启
    │
    ▼
等待节点加入...
    │
    ├── 节点数 < recover_after_nodes → 继续等待
    │
    ├── 节点数 ≥ recover_after_nodes
    │       │
    │       ├── 节点数 = expected_nodes → 立即恢复
    │       │
    │       └── 等待 recover_after_time
    │               │
    │               └── 超时 → 开始恢复
    │
    ▼
开始分片恢复（先恢复 primary，再恢复 replica）
```

---

## 恢复条件详解

### 恢复的触发逻辑

集群恢复需要满足**以下任一条件**（按优先级）：

1. **检查基本限制条件**（必须先满足）：
   - 如果 `recover_after_nodes` 设置：`数据节点数 + master节点数 < recover_after_nodes` → 等待
   - 如果 `recover_after_data_nodes` 设置：`数据节点数 < recover_after_data_nodes` → 等待
   - 如果 `recover_after_master_nodes` 设置：`master节点数 < recover_after_master_nodes` → 等待

2. **检查期望节点条件**（满足任一则可恢复）：
   - 如果设置了 `expected_*` 参数且条件满足 → 立即恢复（跳过超时）
   - 如果设置了 `recover_after_time` → 等待指定时间后恢复

3. **默认行为**（都未设置时）：
   - 立即开始恢复

### 源码验证

根据 `GatewayService.java` 源码中的 `clusterChanged` 方法：

```java
// 恢复被阻止的条件（任一满足则不恢复）
if (recoverAfterNodes != -1 && (nodes.getMasterAndDataNodes().size()) < recoverAfterNodes)
    // 不恢复：总节点数不足
else if (recoverAfterDataNodes != -1 && nodes.getDataNodes().size() < recoverAfterDataNodes)
    // 不恢复：数据节点数不足
else if (recoverAfterMasterNodes != -1 && nodes.getMasterNodes().size() < recoverAfterMasterNodes)
    // 不恢复：主节点数不足
else
    // 检查期望节点：满足则立即恢复，否则等待 recover_after_time
```

---

## ⚠️ 参数弃用说明

| 参数 | 状态 | 替代方案 |
|------|------|----------|
| `recover_after_nodes` | **已弃用** | 使用 `recover_after_data_nodes` + `recover_after_master_nodes` |
| `expected_nodes` | **已弃用** | 使用 `expected_data_nodes` |
| `expected_master_nodes` | **已弃用** | 使用 `expected_data_nodes` 代替 |
| `recover_after_master_nodes` | **已弃用** | 使用 `discovery.zen.minimum_master_nodes` 代替 |

> **为什么弃用？** 新参数提供更精确的控制，区分数据节点和主节点，避免配置歧义。

---

## 配置示例

### 三节点集群（推荐配置）

```yaml
# 推荐：使用数据节点控制而非总节点数
gateway.recover_after_data_nodes: 2
gateway.expected_data_nodes: 3
gateway.recover_after_time: 5m
```

**含义**：2 个数据节点加入后开始计时；5 分钟内第 3 个数据节点也加入则立即恢复；5 分钟后仍只有 2 个数据节点也开始恢复。

### 三节点集群（传统配置 - 已弃用）

```yaml
gateway.recover_after_nodes: 2
gateway.expected_nodes: 3
gateway.recover_after_time: 5m
```

### 角色分离集群（3 master + 5 data）

```yaml
# 按数据节点数控制
gateway.recover_after_data_nodes: 3
gateway.expected_data_nodes: 5
gateway.recover_after_time: 10m
```

### 大规模集群（20+ 节点）

```yaml
# 推荐：分别控制数据节点和主节点
gateway.recover_after_data_nodes: 15
gateway.expected_data_nodes: 20
gateway.recover_after_master_nodes: 2
gateway.recover_after_time: 15m
```

---

## 延伸阅读

- [集群配置]({{< relref "../configuration.md" >}}) — 分片恢复限流（`indices.recovery.max_bytes_per_sec`）等动态设置
- [集群发现]({{< relref "./discovery.md" >}}) — 节点如何发现彼此
- [生产环境部署]({{< relref "/docs/deployment/install-guide/production-env.md" >}}) — 滚动重启流程


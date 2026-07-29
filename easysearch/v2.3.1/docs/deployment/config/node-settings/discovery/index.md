---
title: "集群发现"
date: 0001-01-01
summary: "集群发现配置 #  本页介绍 easysearch.yml 中与节点发现、主节点选举和故障检测相关的配置项。这些都是静态设置，修改后需要重启节点生效。
 discovery.seed_hosts #  discovery.seed_hosts: - 192.168.1.10:9300 - 192.168.1.11:9300 - 192.168.1.12:9300    项目 说明     参数 discovery.seed_hosts   默认值 未设置（自动使用 [&quot;127.0.0.1&quot;, &quot;[::1]&quot;]）   属性 静态   说明 提供集群中候选主节点的地址列表，用于新节点加入时的发现。这是组建多节点集群最关键的配置。未配置时默认连接本地 IPv4 和 IPv6 环回地址    格式说明 #   每个地址格式为 host:port 或 host（省略端口时使用 transport.port 的值，默认 9300） 支持 IP 地址、主机名或域名 只需列出具有 master 角色的节点地址  多种写法：
# YAML 列表 discovery."
---


# 集群发现配置

本页介绍 `easysearch.yml` 中与节点发现、主节点选举和故障检测相关的配置项。这些都是**静态设置**，修改后需要重启节点生效。

---

## discovery.seed_hosts

```yaml
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11:9300
  - 192.168.1.12:9300
```

| 项目 | 说明 |
|------|------|
| **参数** | `discovery.seed_hosts` |
| **默认值** | 未设置（自动使用 `["127.0.0.1", "[::1]"]`） |
| **属性** | 静态 |
| **说明** | 提供集群中候选主节点的地址列表，用于新节点加入时的发现。这是组建多节点集群最关键的配置。未配置时默认连接本地 IPv4 和 IPv6 环回地址 |

### 格式说明

- 每个地址格式为 `host:port` 或 `host`（省略端口时使用 `transport.port` 的值，默认 9300）
- 支持 IP 地址、主机名或域名
- 只需列出具有 master 角色的节点地址

**多种写法**：

```yaml
# YAML 列表
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11:9300
  - 192.168.1.12:9300

# 单行数组
discovery.seed_hosts: ["192.168.1.10:9300", "192.168.1.11:9300"]

# 使用主机名
discovery.seed_hosts:
  - master-1.internal:9300
  - master-2.internal:9300
  - master-3.internal:9300

# 省略端口（使用默认 9300）
discovery.seed_hosts:
  - 192.168.1.10
  - 192.168.1.11
```

### 注意事项

- 列表中不需要包含本机地址，但包含也无妨。
- 不需要列出所有节点，只需要列出 master 候选节点即可。
- 生产环境中**始终使用单播**（seed hosts），不要依赖组播。

---

## cluster.initial_master_nodes

```yaml
cluster.initial_master_nodes:
  - node-1
  - node-2
  - node-3
```

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.initial_master_nodes` |
| **默认值** | `[]`（空列表） |
| **属性** | 静态 |
| **说明** | 新集群**首次启动**时，参与初始主节点选举的节点列表。这个设置仅在集群引导（bootstrap）阶段有效 |

### ⚠️ 重要注意事项

1. **仅用于首次启动**：集群完成首次主节点选举后，应从所有节点的配置中**移除**此设置。
2. **值是节点名称**（`node.name`），不是 IP 地址。
3. **不要在已运行的集群中添加此设置**，否则可能导致脑裂。
4. 列表中的节点必须全部都具有 `master` 角色。

**正确做法**：

```yaml
# 首次启动时配置（使用 node.name 值）
cluster.initial_master_nodes:
  - master-1
  - master-2
  - master-3
```

**也可以使用地址格式**：

```yaml
# 使用 host:transport_port 格式
cluster.initial_master_nodes:
  - 192.168.1.10:9300
  - 192.168.1.11:9300
  - 192.168.1.12:9300
```

---

## discovery.type

```yaml
discovery.type: single-node
```

| 项目 | 说明 |
|------|------|
| **参数** | `discovery.type` |
| **默认值** | `zen`（多节点集群） |
| **属性** | 静态 |
| **说明** | 发现类型。设为 `single-node` 时 Easysearch 形成单节点集群，跳过发现和引导过程 |

| 值 | 行为 |
|------|------|
| `zen` | 默认值，形成多节点集群，通过 `discovery.seed_hosts` 发现其他节点 |
| `single-node` | 单节点模式，该节点自动成为 master，不会与其他节点组成集群 |

**单节点模式适用场景**：

- 本地开发和测试
- CI/CD 环境
- 不需要高可用的小型应用

```yaml
# 单节点模式完整配置
cluster.name: dev-cluster
node.name: dev-node
network.host: 0.0.0.0
discovery.type: single-node
```

---

## 故障检测

Easysearch 通过故障检测（Fault Detection）机制监控集群中其他节点的健康状态。

### discovery.fd.ping_interval

```yaml
discovery.fd.ping_interval: 1s
```

| 项目 | 说明 |
|------|------|
| **参数** | `discovery.fd.ping_interval` |
| **默认值** | `1s` |
| **属性** | 静态 |
| **说明** | 节点间心跳检测的频率。缩短间隔可以更快发现节点故障，但会增加网络开销 |

### discovery.fd.ping_timeout

```yaml
discovery.fd.ping_timeout: 30s
```

| 项目 | 说明 |
|------|------|
| **参数** | `discovery.fd.ping_timeout` |
| **默认值** | `30s` |
| **属性** | 静态 |
| **说明** | 故障检测的超时时间。超过此时间未收到响应则标记为一次失败 |

### discovery.fd.ping_retries

```yaml
discovery.fd.ping_retries: 3
```

| 项目 | 说明 |
|------|------|
| **参数** | `discovery.fd.ping_retries` |
| **默认值** | `3` |
| **属性** | 静态 |
| **说明** | 故障检测的重试次数。连续失败超过此次数后，该节点被判定为离线 |

### 故障检测调优建议

**默认行为**：每 1 秒 ping 一次，超时 30 秒，连续 3 次失败后判定离线。即最快在约 **3 秒**后检测到节点故障。

- **稳定网络环境**：默认值即可。
- **不稳定网络**：适当增大 `ping_timeout` 和 `ping_retries`，避免频繁误判。
- **低延迟场景**（如同机房）：可缩短 `ping_interval` 到 `500ms`，加快故障发现。

```yaml
# 不稳定网络环境
discovery.fd.ping_interval: 2s
discovery.fd.ping_timeout: 60s
discovery.fd.ping_retries: 5

# 低延迟高可用场景
discovery.fd.ping_interval: 500ms
discovery.fd.ping_timeout: 10s
discovery.fd.ping_retries: 3
```

---

## 其他发现参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `discovery.seed_providers` | `settings` | 种子节点提供者类型。`settings` 表示从 `discovery.seed_hosts` 读取；`file` 表示从 `$ES_PATH_CONF/unicast_hosts.txt` 动态加载（无需重启即可更新节点列表） |
| `discovery.cluster_formation_warning_timeout` | `10s` | 集群形成超时后输出警告日志的等待时间 |

---

## 配置示例

### 分布式集群模式

3 个独立 Master 节点 + N 个 Data 节点的标准生产架构：

**Master 节点配置**：

```yaml
cluster.name: prod-cluster
node.name: master-1
node.roles: [master]
network.host: 192.168.1.10
http.port: 9200
transport.port: 9300
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11:9300
  - 192.168.1.12:9300
# 仅首次启动时配置，集群建立后移除
cluster.initial_master_nodes:
  - master-1
  - master-2
  - master-3
```

**Data 节点配置**：

```yaml
cluster.name: prod-cluster
node.name: data-1
node.roles: [data, ingest]
network.host: 192.168.1.20
http.port: 9200
transport.port: 9300
# Data 节点只需指向 Master 节点
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11:9300
  - 192.168.1.12:9300
# Data 节点不需要配置 cluster.initial_master_nodes
```

> Master 使用 3 个就够了，数据节点可以根据需要动态增加。

### 单节点开发模式

```yaml
cluster.name: dev-cluster
node.name: dev-node
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300
discovery.type: single-node
```

---

## 延伸阅读

- [集群与节点]({{< relref "./cluster-node.md" >}}) — 节点角色与命名
- [网络配置]({{< relref "./network.md" >}}) — 绑定地址与端口
- [网关与恢复]({{< relref "./gateway.md" >}}) — 集群重启后的分片恢复行为
- [集群管理]({{< relref "/docs/operations/cluster-admin/cluster.md" >}}) — 集群管理操作指南


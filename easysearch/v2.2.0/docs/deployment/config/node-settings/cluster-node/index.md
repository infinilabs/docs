---
title: "集群与节点"
date: 0001-01-01
summary: "集群与节点配置 #  本页介绍 easysearch.yml 中与集群身份和节点角色相关的配置项。这些都是静态设置，修改后需要重启节点生效。
 集群名称 #  cluster.name: easysearch    项目 说明     参数 cluster.name   默认值 easysearch   属性 静态   说明 集群的唯一名称。所有节点必须配置相同的集群名称才能组成一个集群。不同环境（开发、测试、生产）应使用不同的集群名称以防止节点误加入    注意事项：
 集群名称不能包含冒号（:），建议仅使用小写字母、数字和连字符。 集群名称一旦投入使用后不建议更改，因为它会影响数据目录结构。 同一网络中如果有多个集群，务必确保集群名称各不相同。  示例：
# 开发环境 cluster.name: dev-cluster # 生产环境 cluster.name: prod-search  节点名称 #  node.name: node-1    项目 说明     参数 node.name   默认值 主机名（hostname）   属性 静态   说明 节点在集群中的唯一标识符。如果未显式设置，默认使用机器主机名。用于日志输出、集群状态显示、API 返回信息中。建议使用有意义的名称便于运维定位    命名建议："
---


# 集群与节点配置

本页介绍 `easysearch.yml` 中与集群身份和节点角色相关的配置项。这些都是**静态设置**，修改后需要重启节点生效。

---

## 集群名称

```yaml
cluster.name: easysearch
```

| 项目 | 说明 |
|------|------|
| **参数** | `cluster.name` |
| **默认值** | `easysearch` |
| **属性** | 静态 |
| **说明** | 集群的唯一名称。所有节点必须配置相同的集群名称才能组成一个集群。不同环境（开发、测试、生产）应使用不同的集群名称以防止节点误加入 |

**注意事项**：

- 集群名称**不能包含冒号（`:`）**，建议仅使用小写字母、数字和连字符。
- 集群名称一旦投入使用后不建议更改，因为它会影响数据目录结构。
- 同一网络中如果有多个集群，务必确保集群名称各不相同。

**示例**：

```yaml
# 开发环境
cluster.name: dev-cluster

# 生产环境
cluster.name: prod-search
```

---

## 节点名称

```yaml
node.name: node-1
```

| 项目 | 说明 |
|------|------|
| **参数** | `node.name` |
| **默认值** | 主机名（hostname） |
| **属性** | 静态 |
| **说明** | 节点在集群中的唯一标识符。如果未显式设置，默认使用机器主机名。用于日志输出、集群状态显示、API 返回信息中。建议使用有意义的名称便于运维定位 |

**命名建议**：

- 采用统一的命名规则，例如 `<角色>-<编号>` 或 `<机房>-<角色>-<编号>`
- 名称在集群中必须唯一

**示例**：

```yaml
# 按角色命名
node.name: master-1
node.name: data-hot-1
node.name: data-warm-2

# 按机房命名
node.name: bj-data-01
node.name: sh-master-02

# 使用环境变量
node.name: ${HOSTNAME}
```

---

## 节点角色

```yaml
node.roles: [master, data, ingest, remote_cluster_client]
```

| 项目 | 说明 |
|------|------|
| **参数** | `node.roles` |
| **默认值** | `[master, data, ingest, remote_cluster_client]` |
| **属性** | 静态 |
| **说明** | 决定节点在集群中承担的功能。默认情况下节点拥有所有角色 |

### 可选角色

| 角色 | 缩写 | 功能 | 适用场景 |
|------|------|------|----------|
| `master` | `m` | 参与主节点选举，管理集群状态（索引创建/删除、分片分配等） | 每个集群建议 3 个专用 master 节点 |
| `data` | `d` | 存储索引数据，执行 CRUD、搜索和聚合操作 | 主要计算与存储工作负载 |
| `ingest` | `i` | 执行 ingest pipeline（文档预处理） | 有复杂预处理需求时可独立部署 |
| `remote_cluster_client` | `r` | 充当跨集群搜索/复制的客户端 | 需要跨集群查询时启用 |
| `search` | `s` | 执行搜索操作 | 专用搜索节点场景 |

> 缩写字母会在 `GET _cat/nodes` 输出的 `node.role` 列中显示。

### 旧版配置兼容

以下旧版布尔配置已废弃，建议迁移到 `node.roles`：

| 旧配置 | 等价 `node.roles` 写法 |
|--------|------------------------|
| `node.master: true` | `node.roles` 中包含 `master` |
| `node.data: true` | `node.roles` 中包含 `data` |
| `node.ingest: true` | `node.roles` 中包含 `ingest` |
| `node.master: false` + `node.data: false` | `node.roles: []`（协调节点） |

### 架构建议

**小规模集群**（3 节点）— 所有节点同时承担所有角色：

```yaml
# 所有节点使用相同配置
node.roles: [master, data, ingest]
```

**中大规模集群**（10+ 节点）— 角色分离：

```yaml
# 专用 Master 节点（3 个，不存储数据，资源占用低）
node.roles: [master]

# 专用 Data 节点（可水平扩展）
node.roles: [data, ingest]

# 专用协调节点（无数据、无 master 资格，仅做请求路由和结果聚合）
node.roles: []
```

**冷热分层集群** — 搭配 `node.attr.temp` 使用：

```yaml
# 热节点（SSD，接收实时写入）
node.roles: [data]
node.attr.temp: hot

# 温节点（大容量 HDD，存储历史数据）
node.roles: [data]
node.attr.temp: warm

# 冷节点（归档）
node.roles: [data]
node.attr.temp: cold
```

---

## 自定义节点属性

```yaml
node.attr.zone: cn-hangzhou-a
node.attr.rack: rack-01
node.attr.temp: hot
```

| 项目 | 说明 |
|------|------|
| **参数** | `node.attr.<key>` |
| **默认值** | 无 |
| **属性** | 静态 |
| **说明** | 自定义的键值对属性，附加到节点上。可用于分片分配感知（Shard Allocation Awareness）和分片分配过滤 |

### 常用场景

#### 可用区感知

确保主分片和副本分布在不同的可用区，提高容灾能力：

```yaml
# 节点配置（easysearch.yml）
node.attr.zone: cn-hangzhou-a
```

搭配集群设置 API 启用分配感知：

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "zone"
  }
}
```

#### 机架感知

```yaml
node.attr.rack: rack-01
```

#### 冷热分层

```yaml
# 热节点
node.attr.temp: hot

# 温节点
node.attr.temp: warm
```

搭配索引生命周期管理或手动分配：

```json
PUT /old-logs-*/_settings
{
  "index.routing.allocation.require.temp": "warm"
}
```

### 查看节点属性

```bash
GET _cat/nodeattrs?v
```

---

## 系统启动配置

### bootstrap.system_call_filter

```yaml
bootstrap.system_call_filter: false
```

| 项目 | 说明 |
|------|------|
| **参数** | `bootstrap.system_call_filter` |
| **默认值** | `true` |
| **属性** | 静态 |
| **说明** | 是否启用 Linux 系统调用过滤（seccomp）。设为 `false` 时禁用系统调用过滤。在某些运行环境（如 Docker、虚拟机、某些云平台）中可能需要禁用以避免启动失败 |

**禁用场景**：

- 在不支持 seccomp 的容器环境中运行
- 某些云计算平台有限制
- 开发和测试环境

> 生产环境建议保持启用（`true`）以增强安全性。

---

## 完整配置示例

### 三节点通用集群

```yaml
# ---- node-1 ----
cluster.name: prod-cluster
node.name: node-1
node.roles: [master, data, ingest]
node.attr.zone: zone-a

# ---- node-2 ----
cluster.name: prod-cluster
node.name: node-2
node.roles: [master, data, ingest]
node.attr.zone: zone-b

# ---- node-3 ----
cluster.name: prod-cluster
node.name: node-3
node.roles: [master, data, ingest]
node.attr.zone: zone-a
```

### 角色分离集群

```yaml
# ---- master-1 ----
cluster.name: prod-cluster
node.name: master-1
node.roles: [master]

# ---- data-hot-1 ----
cluster.name: prod-cluster
node.name: data-hot-1
node.roles: [data, ingest]
node.attr.temp: hot

# ---- data-warm-1 ----
cluster.name: prod-cluster
node.name: data-warm-1
node.roles: [data]
node.attr.temp: warm
```

---

## 延伸阅读

- [网络配置]({{< relref "./network.md" >}}) — 绑定地址与端口
- [集群发现]({{< relref "./discovery.md" >}}) — 节点如何发现彼此组成集群
- [集群配置]({{< relref "../configuration.md" >}}) — 分片分配感知等动态设置


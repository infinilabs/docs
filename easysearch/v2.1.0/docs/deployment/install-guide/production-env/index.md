---
title: "生产环境部署"
date: 0001-01-01
summary: "生产环境部署指南 #  本文提供 Easysearch 生产环境部署的完整建议，涵盖硬件选型、节点规划、高可用架构与上线前检查清单。
硬件推荐 #     组件 CPU 内存 JVM 堆 磁盘 高可用实例数     Easysearch 16 核+ 64 GB+ 31 GB SSD ≥ 3   INFINI Console 8 核 16 GB — ≥ 50 GB 1   INFINI Gateway 8 核 16 GB — ≥ 50 GB ≥ 2     详见 硬件配置。
 磁盘选型 #   生产环境：强烈建议使用本地 SSD（NVMe 优先）。随机 IOPS 和吞吐量直接影响索引写入与查询延迟。 避免网络存储：NFS / CIFS 不适用于 Easysearch 数据目录。云环境可使用高性能云盘（如阿里云 ESSD、AWS gp3/io2）。 容量估算：原始数据量 × (1 + 副本数) × 1."
---


# 生产环境部署指南

本文提供 Easysearch 生产环境部署的完整建议，涵盖硬件选型、节点规划、高可用架构与上线前检查清单。

## 硬件推荐

| 组件 | CPU | 内存 | JVM 堆 | 磁盘 | 高可用实例数 |
|:-----|:---:|:----:|:------:|:----:|:----------:|
| Easysearch | 16 核+ | 64 GB+ | 31 GB | SSD | ≥ 3 |
| INFINI Console | 8 核 | 16 GB | — | ≥ 50 GB | 1 |
| INFINI Gateway | 8 核 | 16 GB | — | ≥ 50 GB | ≥ 2 |

> 详见 [硬件配置]({{< relref "/docs/deployment/config/hardware.md" >}})。

### 磁盘选型

- **生产环境**：强烈建议使用本地 SSD（NVMe 优先）。随机 IOPS 和吞吐量直接影响索引写入与查询延迟。
- **避免网络存储**：NFS / CIFS 不适用于 Easysearch 数据目录。云环境可使用高性能云盘（如阿里云 ESSD、AWS gp3/io2）。
- **容量估算**：原始数据量 × (1 + 副本数) × 1.1（段合并预留），再预留 20% 空间用于日常运维操作。

### 内存分配

- JVM 堆设置为物理内存的 **50%**，但**不超过 31 GB**（压缩指针阈值）。
- 剩余内存留给操作系统文件缓存，Lucene 段读取严重依赖页缓存。
- 示例：64 GB 机器 → `-Xms31g -Xmx31g`。

```bash
# config/jvm.options
-Xms31g
-Xmx31g
```

## 节点角色规划

Easysearch 支持以下节点角色，生产环境建议分角色部署：

| 角色 | 说明 | 建议数量 |
|------|------|:--------:|
| `master` | 集群状态管理、索引元数据 | 3（专用） |
| `data` | 索引和搜索数据 | 按数据量扩展 |
| `ingest` | 摄取管道预处理 | 按写入量可选 |
| `coordinating` | 查询路由与结果归并 | 高并发场景可选 |

### 小集群（3 节点起步）

3 个节点同时承担 master + data 角色，适合数据量 < 1 TB 的场景：

```yaml
# easysearch.yml（每个节点）
cluster.name: prod-cluster
node.name: node-1
node.roles: [master, data]
network.host: 0.0.0.0
discovery.seed_hosts: ["10.0.1.1", "10.0.1.2", "10.0.1.3"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

### 中大集群（6+ 节点）

master 与 data 分离，可独立扩容数据层：

```
3 × master（专用，4C16G 即可）
N × data（16C64G + SSD，按需扩展）
2 × coordinating（可选，高并发查询场景）
```

## 系统调优

在启动 Easysearch 之前，请完成以下操作系统调优：

```bash
# 1. 文件描述符
ulimit -n 65535

# 2. 虚拟内存
sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" >> /etc/sysctl.conf

# 3. 禁用 swap
swapoff -a
# 或在 easysearch.yml 中设置：
# bootstrap.memory_lock: true
```

> 完整参数见 [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})。

## 安全配置

Easysearch **默认启用安全功能**，首次部署需初始化：

```bash
# 初始化安全模块（生成证书 + admin 密码）
bin/initialize.sh -s
# 密码将在终端输出中显示，请务必记住
# 也可预先设置环境变量：export EASYSEARCH_INITIAL_ADMIN_PASSWORD="YourSecurePassword"
```

生产环境建议：

- 使用企业 CA 签发的证书替换自签名证书
- 启用节点间 TLS 传输加密
- 配置 LDAP / AD 对接（如有）
- 设置最小权限的角色与用户

> 详见 [安全配置]({{< relref "/docs/operations/security/" >}})。

## 备份策略

生产环境**必须**配置快照备份：

```
# 注册快照仓库
PUT _snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mnt/backup/easysearch"
  }
}

# 创建快照
PUT _snapshot/my_backup/snapshot_1
```

建议：

- 每日自动快照，保留 7～30 天
- 备份存储与数据存储物理隔离
- 定期验证恢复流程

> 详见 [备份与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}})。

## 滚动重启

当需要对集群进行维护（升级版本、更换硬件、修改配置等）时，应使用**滚动重启**方式逐节点操作，避免集群停机。

### 操作步骤

1. **暂停分片分配**（避免节点下线后触发不必要的数据迁移）

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
```

2. **执行 synced flush**（减少恢复时间，可选）

```
POST /_flush/synced
```

3. **停止并维护目标节点**

```bash
# 停止节点
kill $(cat /data/easysearch/pid)

# 执行维护操作（升级、配置修改等）
# ...

# 重新启动节点
su easysearch -c "/data/easysearch/bin/easysearch -d -p pid"
```

4. **等待节点重新加入集群**

```bash
# 确认节点已加入
curl -ku admin:YOUR_PASSWORD "https://localhost:9200/_cat/nodes?v"
```

5. **恢复分片分配**

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```

6. **等待集群恢复 green 状态**

```bash
curl -ku admin:YOUR_PASSWORD "https://localhost:9200/_cluster/health?wait_for_status=green&timeout=120s"
```

7. **重复步骤 1-6**，直到所有节点维护完毕（每次只操作一个节点）。

> **注意**：始终先处理数据节点，最后处理当前 master 节点。如果节点在短时间内重新加入，Easysearch 会自动使用本地数据恢复分片，避免完整的数据复制。

## 上线前检查清单

| 检查项 | 说明 |
|--------|------|
| 集群状态 | `GET _cluster/health` 返回 `green` |
| 节点数量 | `GET _cat/nodes` 确认所有节点已加入 |
| 分片分配 | 无 unassigned 分片 |
| JVM 堆 | 不超过 31 GB，`-Xms` = `-Xmx` |
| 文件描述符 | `GET _nodes/stats/process` 确认 `max_file_descriptors` ≥ 65535 |
| swap | 已禁用或 `memory_lock: true` |
| 快照 | 已配置并测试恢复 |
| 监控 | INFINI Console 已连接，告警已配置 |
| 安全 | TLS 启用，默认密码已更改 |

## 延伸阅读

- [分布式集群部署]({{< relref "./cluster.md" >}})
- [硬件配置]({{< relref "/docs/deployment/config/hardware.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})
- [容量规划]({{< relref "/docs/best-practices/index-design.md" >}})
- [扩缩容与分片]({{< relref "/docs/operations/cluster-admin/cluster" >}})


---
title: "路径配置"
date: 0001-01-01
summary: "路径配置 #  本页介绍 easysearch.yml 中与文件存储路径相关的配置项。这些都是静态设置，修改后需要重启节点生效。
 path.data #  path.data: /data/easysearch/data    项目 说明     参数 path.data   默认值 $ES_HOME/data   属性 静态   说明 索引数据的存储目录。Easysearch 的所有分片数据（Lucene 段文件、translog）都存储在此目录下。这是磁盘 I/O 最密集的路径    注意事项 #   不要使用默认路径：默认路径在安装目录内（$ES_HOME/data），升级时可能被覆盖。生产环境必须设置独立路径。 目录权限：运行 Easysearch 的用户必须对该目录拥有读写权限。 磁盘选择：建议使用 SSD 存储，可显著提升索引和查询性能。 不要在多个节点之间共享：每个节点的 path.data 必须是独立的目录。  多路径（JBOD） #  支持配置多个数据路径，实现 JBOD（Just a Bunch of Disks）条带化，将分片分散到多块磁盘上：
# YAML 列表形式 path.data: - /data1/easysearch - /data2/easysearch - /data3/easysearch 多路径行为说明："
---


# 路径配置

本页介绍 `easysearch.yml` 中与文件存储路径相关的配置项。这些都是**静态设置**，修改后需要重启节点生效。

---

## path.data

```yaml
path.data: /data/easysearch/data
```

| 项目 | 说明 |
|------|------|
| **参数** | `path.data` |
| **默认值** | `$ES_HOME/data` |
| **属性** | 静态 |
| **说明** | 索引数据的存储目录。Easysearch 的所有分片数据（Lucene 段文件、translog）都存储在此目录下。这是磁盘 I/O 最密集的路径 |

### 注意事项

- **不要使用默认路径**：默认路径在安装目录内（`$ES_HOME/data`），升级时可能被覆盖。生产环境必须设置独立路径。
- **目录权限**：运行 Easysearch 的用户必须对该目录拥有读写权限。
- **磁盘选择**：建议使用 SSD 存储，可显著提升索引和查询性能。
- **不要在多个节点之间共享**：每个节点的 `path.data` 必须是独立的目录。

### 多路径（JBOD）

支持配置多个数据路径，实现 JBOD（Just a Bunch of Disks）条带化，将分片分散到多块磁盘上：

```yaml
# YAML 列表形式
path.data:
  - /data1/easysearch
  - /data2/easysearch
  - /data3/easysearch
```

**多路径行为说明**：

- Easysearch 会将不同的**分片**分配到不同的路径上。
- 单个分片的所有文件始终存储在同一路径下，不会跨磁盘。
- 磁盘空间的均衡取决于分片的大小和数量。
- 如果其中一块磁盘故障，该磁盘上的所有分片都会不可用，但由于有副本机制，数据不会丢失。

> 如果需要磁盘级别的冗余，建议使用 RAID 或云盘。参见 [RAID 配置]({{< relref "/docs/deployment/advanced-config/raid.md" >}})。

---

## path.logs

```yaml
path.logs: /data/easysearch/logs
```

| 项目 | 说明 |
|------|------|
| **参数** | `path.logs` |
| **默认值** | `$ES_HOME/logs` |
| **属性** | 静态 |
| **说明** | 日志文件的存储目录。包括主日志、慢日志、GC 日志等 |

### 注意事项

- 同样不建议使用默认路径。
- 日志目录应与数据目录分开，避免日志写入影响数据磁盘 I/O。
- 确保有足够的磁盘空间存储日志，尤其在开启 DEBUG 日志时。

### 日志文件说明

| 文件 | 说明 |
|------|------|
| `<cluster.name>.log` | 主日志文件 |
| `<cluster.name>_server.json` | JSON 格式日志（便于日志采集） |
| `<cluster.name>_deprecation.log` | API 弃用警告日志 |
| `<cluster.name>_slowlog.log` | 慢查询/慢索引日志 |
| `gc.log` | JVM GC 日志 |

---

## path.repo

```yaml
path.repo: ["/mnt/backup/easysearch"]
```

| 项目 | 说明 |
|------|------|
| **参数** | `path.repo` |
| **默认值** | 未设置 |
| **属性** | 静态 |
| **说明** | 快照仓库的共享文件系统路径。使用 `fs` 类型的快照仓库时必须配置此项 |

### 注意事项

- 该路径必须在**所有 master 和 data 节点**上都可访问（通常通过 NFS 共享挂载）。
- 支持多个路径（YAML 列表形式）。
- 创建快照仓库时指定的 `location` 必须是 `path.repo` 下的子目录。

### 使用示例

**步骤 1：配置 path.repo（所有节点）**

```yaml
path.repo:
  - /mnt/nfs/backup
  - /mnt/nfs/archive
```

**步骤 2：创建快照仓库**

```json
PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mnt/nfs/backup/my_repo"
  }
}
```

---

## path.shared_data

```yaml
path.shared_data: /data/easysearch/shared_data
```

| 项目 | 说明 |
|------|------|
| **参数** | `path.shared_data` |
| **默认值** | 未设置 |
| **属性** | 静态 |
| **说明** | 集群内所有节点共享的数据目录（通常用于共享库、字典等）。可选配置 |

### 注意事项

- 该路径应该在所有节点上都可访问，通常通过 NFS 挂载。
- 主要用于存储需要在集群间共享的资源（如分析器字典、插件数据等）。

---

## 完整配置示例

### 生产环境（单磁盘）

```yaml
path.data: /data/easysearch/data
path.logs: /var/log/easysearch
path.repo: ["/mnt/nfs/easysearch-backup"]
```

### 生产环境（多磁盘 JBOD）

```yaml
path.data:
  - /data1/easysearch
  - /data2/easysearch
  - /data3/easysearch
  - /data4/easysearch
path.logs: /var/log/easysearch
path.repo: ["/mnt/nfs/backup"]
```

### 开发/测试环境

```yaml
# 使用默认路径即可，无需显式配置
# path.data: data
# path.logs: logs
```

---

## 延伸阅读

- [硬件配置]({{< relref "../hardware.md" >}}) — 磁盘选型建议（SSD vs HDD、JBOD vs RAID）
- [RAID 配置]({{< relref "/docs/deployment/advanced-config/raid.md" >}}) — JBOD 多路径 vs. RAID 方案对比
- [日志配置]({{< relref "./logging.md" >}}) — log4j2.properties 日志格式与级别


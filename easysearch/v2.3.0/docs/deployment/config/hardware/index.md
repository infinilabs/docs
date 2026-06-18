---
title: "硬件配置"
date: 0001-01-01
summary: "生产环境硬件配置推荐 #  在生产环境部署 Easysearch 时，高可用性（HA）是必须满足的核心要求。为实现完整的 HA 保障，您至少需要部署 3 个节点组成 Easysearch 集群。为获得最佳运维体验，建议配合使用 INFINI Console 和 Gateway，它们提供集群监控、告警和运维管理等完整功能，可大幅提升日常运维效率。
推荐配置 #     产品 CPU 内存 JVM 堆 磁盘 高可用实例数     Easysearch 16 核+ 64 GB+ 31 GB SSD ≥ 3   INFINI Console 8 核 16 GB — ≥ 50 GB (HDD/SSD) 1   INFINI Gateway 8 核 16 GB — ≥ 50 GB (HDD/SSD) ≥ 2     对于低负载集群或测试环境，可适当降低配置标准，但需确保满足基础性能需求。测试环境最低配置参见 测试环境部署。"
---

# 生产环境硬件配置推荐

在生产环境部署 Easysearch 时，高可用性（HA）是必须满足的核心要求。为实现完整的 HA 保障，您至少需要部署 **3 个节点**组成 Easysearch 集群。为获得最佳运维体验，建议配合使用 INFINI Console 和 Gateway，它们提供集群监控、告警和运维管理等完整功能，可大幅提升日常运维效率。

## 推荐配置

| 产品 | CPU | 内存 | JVM 堆 | 磁盘 | 高可用实例数 |
|:-----|:---:|:----:|:------:|:----:|:----------:|
| Easysearch | 16 核+ | 64 GB+ | 31 GB | SSD | ≥ 3 |
| INFINI Console | 8 核 | 16 GB | — | ≥ 50 GB (HDD/SSD) | 1 |
| INFINI Gateway | 8 核 | 16 GB | — | ≥ 50 GB (HDD/SSD) | ≥ 2 |

> 对于低负载集群或测试环境，可适当降低配置标准，但需确保满足基础性能需求。测试环境最低配置参见 [测试环境部署]({{< relref "/docs/deployment/install-guide/test-env.md" >}})。

## 内存

内存通常是 Easysearch 最重要的资源。充足的内存可以显著降低查询延迟并提高索引吞吐。

### JVM 堆内存

- 将 JVM 堆设置为物理内存的 **50%**，但**不超过 31 GB**。
- 超过 31 GB 后，JVM 将无法使用[压缩指针（Compressed Oops）](https://wiki.openjdk.org/display/HotSpot/CompressedOops)，实际可用堆反而减少。
- 始终保持 `-Xms` 和 `-Xmx` 相同，避免运行时堆调整带来的 GC 停顿。

```bash
# config/jvm.options 示例（64 GB 机器）
-Xms31g
-Xmx31g
```

### 操作系统文件缓存

**剩余的 50% 内存留给操作系统**。Lucene 的段文件依赖操作系统页缓存（Page Cache）来加速读取。如果把所有内存都给了 JVM，段读取将退化为磁盘 I/O，搜索性能会大幅下降。

### 大内存机器策略

如果机器内存 ≥ 128 GB，可考虑在同一台机器上运行多个 Easysearch 实例（每个实例 31 GB 堆），配合 [NUMA 绑定]({{< relref "/docs/deployment/advanced-config/numa.md" >}})获得最佳性能。

## CPU

Easysearch 对 CPU 的需求取决于查询复杂度和写入压力：

- **更多核心优于更高主频**：Easysearch 的索引、搜索、段合并等操作高度并行化，多核心可以并行处理更多任务。
- 现代多核 CPU（如 16 核、32 核）完全可以满足大多数场景。
- 超高主频的单核处理器并不比普通主频的多核处理器更有优势。

| 节点角色 | CPU 建议 |
|----------|----------|
| Master（专用） | 4 核即可，主要消耗在集群状态管理 |
| Data | 16 核+，索引和搜索都需要大量计算 |
| Coordinating | 8 核+，主要消耗在查询结果归并 |

## 磁盘

磁盘是搜索引擎性能的关键瓶颈之一。Easysearch 的索引写入、段合并、translog 刷写以及搜索读取都依赖磁盘 I/O。

### SSD vs. HDD

| 指标 | SATA SSD | NVMe SSD | HDD |
|------|:--------:|:--------:|:---:|
| 随机读 IOPS | ~90K | ~500K+ | ~200 |
| 随机写 IOPS | ~50K | ~200K+ | ~200 |
| 适用场景 | 通用生产 | 高性能生产 | 仅冷数据/归档 |

- **生产环境强烈建议使用 SSD**，NVMe 优先。
- 如果使用 SSD，应将 I/O 调度器设置为 `noop` 或 `none`。
- **避免使用网络存储**（NFS / CIFS / SMB），这类存储不适合 Easysearch 的高频随机读写。
- 云环境可使用高性能云盘（如阿里云 ESSD、AWS gp3/io2、腾讯云增强型 SSD）。

> 详细 NVMe 配置参见 [NVMe 配置指南]({{< relref "/docs/deployment/advanced-config/nvme.md" >}})。

### 磁盘挂载建议

- 数据盘**单独挂载**到 `/data`，不要与系统盘混用。
- 使用 `xfs` 或 `ext4` 文件系统。
- 关闭 `atime`：`mount -o noatime /dev/vdb /data`。
- 多块磁盘使用 [JBOD 多路径]({{< relref "/docs/deployment/advanced-config/raid.md" >}})，而非 RAID。

### 容量估算

```
所需磁盘空间 = 原始数据量 × (1 + 副本数) × 1.1（段合并预留）
实际分配空间 = 所需磁盘空间 × 1.2（日常运维预留）
```

示例：100 GB 原始数据，1 个副本 → 100 × 2 × 1.1 × 1.2 ≈ **264 GB**。

## 网络

- 集群节点间建议使用**千兆以上**网络，万兆网卡更佳。
- 低延迟网络对集群稳定性至关重要，**避免跨数据中心部署单个集群**。
- 如有跨地域需求，应使用 [跨集群复制]({{< relref "/docs/operations/cluster-admin/ccr" >}})。
- 节点间通信延迟应 < 1 ms（同机房），跨可用区延迟应 < 5 ms。

## 机器数量与规格

**推荐中等配置的机器，避免极端**：

| 方案 | 说明 |
|------|------|
| ❌ 少量超大机器 | 单节点故障影响太大，恢复时间长 |
| ❌ 大量低配机器 | 管理复杂度高，资源利用率低 |
| ✅ 适量中高配机器 | 平衡成本、性能和容错能力 |

建议的"甜点"配置：**16C64G + NVMe SSD（500 GB ~ 2 TB）**。

## 延伸阅读

- [测试环境部署]({{< relref "/docs/deployment/install-guide/test-env.md" >}})
- [生产环境部署]({{< relref "/docs/deployment/install-guide/production-env.md" >}})
- [NUMA 配置]({{< relref "/docs/deployment/advanced-config/numa.md" >}})
- [NVMe 配置]({{< relref "/docs/deployment/advanced-config/nvme.md" >}})
- [RAID 配置]({{< relref "/docs/deployment/advanced-config/raid.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})

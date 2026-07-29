---
title: "RAID 配置"
date: 0001-01-01
summary: "RAID 配置指南 #  本文讨论 Easysearch 环境中 RAID 的选型与配置建议。
RAID 与 Easysearch 的关系 #  Easysearch 自身通过副本机制实现数据冗余。因此 RAID 在 Easysearch 场景中的定位与传统数据库不同：
   方案 数据冗余 性能 适用场景     无 RAID + 副本 由 ES 副本保障 最高 推荐方案   RAID-0 无 高 多 SATA 盘聚合（有副本前提下）   RAID-1 镜像 中 单节点无副本（不推荐）   RAID-5/6 校验 较低 不推荐（写放大严重）   RAID-10 镜像+条带 中高 预算充足且需要本地冗余     核心建议：优先使用 Easysearch 副本，而非 RAID 来保障数据安全。"
---


# RAID 配置指南

本文讨论 Easysearch 环境中 RAID 的选型与配置建议。

## RAID 与 Easysearch 的关系

Easysearch 自身通过**副本机制**实现数据冗余。因此 RAID 在 Easysearch 场景中的定位与传统数据库不同：

| 方案 | 数据冗余 | 性能 | 适用场景 |
|------|:--------:|:----:|----------|
| 无 RAID + 副本 | 由 ES 副本保障 | 最高 | **推荐方案** |
| RAID-0 | 无 | 高 | 多 SATA 盘聚合（有副本前提下） |
| RAID-1 | 镜像 | 中 | 单节点无副本（不推荐） |
| RAID-5/6 | 校验 | 较低 | **不推荐**（写放大严重） |
| RAID-10 | 镜像+条带 | 中高 | 预算充足且需要本地冗余 |

> **核心建议**：优先使用 Easysearch 副本，而非 RAID 来保障数据安全。

## 推荐方案：JBOD（多路径）

JBOD（Just a Bunch of Disks）是 Easysearch 场景的最佳方案——每块盘独立挂载，通过多路径配置：

```yaml
# easysearch.yml
path.data:
  - /data1/easysearch
  - /data2/easysearch
  - /data3/easysearch
  - /data4/easysearch
```

优势：
- 无 RAID 写放大
- 单盘故障只影响该盘上的分片（副本在其他节点）
- 最大化利用磁盘吞吐

## 如果必须用 RAID

### RAID-0（条带化）

```bash
# 创建 RAID-0（2 块盘条带化）
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc

# 格式化
mkfs.xfs -f /dev/md0

# 挂载
mount -o noatime /dev/md0 /data/easysearch
```

- 适用于：多块 SATA SSD，需要聚合带宽
- 风险：任意一块盘故障，整个阵列数据丢失（依赖 ES 副本恢复）

### RAID-10（镜像+条带）

```bash
# 创建 RAID-10（4 块盘）
mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

- 适用于：单节点无法设置副本的特殊场景
- 代价：50% 磁盘空间用于镜像

## 不推荐 RAID-5/6

RAID-5/6 的奇偶校验计算在 Easysearch 的高随机写入场景下会导致严重的**写放大**：

- 每次写入需要读-改-写校验块
- 段合并时写放大尤其明显
- 重建时间长（TB 级别可能需要数小时）

## 监控

```bash
# 查看 RAID 状态
cat /proc/mdstat

# 或使用 mdadm
mdadm --detail /dev/md0
```

## 延伸阅读

- [NVMe 配置]({{< relref "./nvme.md" >}})
- [生产环境部署]({{< relref "/docs/deployment/install-guide/production-env.md" >}})


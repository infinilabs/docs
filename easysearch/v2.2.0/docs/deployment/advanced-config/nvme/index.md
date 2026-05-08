---
title: "NVMe 配置"
date: 0001-01-01
summary: "NVMe 配置指南 #  NVMe SSD 提供极高的 IOPS 和带宽，是 Easysearch 生产环境的首选存储。
为什么选择 NVMe #     指标 SATA SSD NVMe SSD     随机读 IOPS ~90K ~500K+   随机写 IOPS ~50K ~200K+   顺序读带宽 ~550 MB/s ~3,500 MB/s   延迟 ~100us ~20us    Easysearch 的段合并、translog 写入、倒排索引读取都受益于低延迟和高 IOPS。
磁盘格式化与挂载 #  # 查看 NVMe 设备 lsblk | grep nvme # 格式化（推荐 xfs 或 ext4） mkfs."
---


# NVMe 配置指南

NVMe SSD 提供极高的 IOPS 和带宽，是 Easysearch 生产环境的首选存储。

## 为什么选择 NVMe

| 指标 | SATA SSD | NVMe SSD |
|------|:--------:|:--------:|
| 随机读 IOPS | ~90K | ~500K+ |
| 随机写 IOPS | ~50K | ~200K+ |
| 顺序读带宽 | ~550 MB/s | ~3,500 MB/s |
| 延迟 | ~100us | ~20us |

Easysearch 的段合并、translog 写入、倒排索引读取都受益于低延迟和高 IOPS。

## 磁盘格式化与挂载

```bash
# 查看 NVMe 设备
lsblk | grep nvme

# 格式化（推荐 xfs 或 ext4）
mkfs.xfs -f /dev/nvme0n1

# 挂载（关键：noatime 减少元数据写入）
mkdir -p /data/easysearch
mount -o noatime,nodiratime /dev/nvme0n1 /data/easysearch

# 持久化
echo "/dev/nvme0n1 /data/easysearch xfs noatime,nodiratime 0 0" >> /etc/fstab
```

### 文件系统选择

| 文件系统 | 优势 | 适用场景 |
|----------|------|----------|
| **xfs** | 大文件处理优秀、并行 IO 好 | 大索引、高吞吐 |
| **ext4** | 稳定成熟、小文件性能好 | 通用场景 |

## IO 调度器

NVMe 建议使用 `none` 调度器：

```bash
# 查看当前调度器
cat /sys/block/nvme0n1/queue/scheduler

# 设置为 none
echo none > /sys/block/nvme0n1/queue/scheduler

# 持久化（udev 规则）
echo 'ACTION=="add|change", KERNEL=="nvme*", ATTR{queue/scheduler}="none"' \
  > /etc/udev/rules.d/60-nvme-scheduler.rules
```

## 多盘配置

多块 NVMe 时，配置多个数据路径实现条带化：

```yaml
# easysearch.yml
path.data:
  - /data1/easysearch
  - /data2/easysearch
```

Easysearch 会将分片分布到不同路径，自动利用多盘并行 IO。

> 不建议在多 NVMe 上再做软 RAID-0，直接用多路径更简单。

## 健康监控

```bash
# 查看 NVMe SMART 信息
nvme smart-log /dev/nvme0n1

# 关注指标：
# - percentage_used: 磨损百分比
# - data_units_written: 总写入量
# - media_errors: 介质错误数
```

## 云环境 NVMe

| 云厂商 | 本地 NVMe 实例 | 注意事项 |
|--------|---------------|----------|
| 阿里云 | i 系列 | 实例释放后数据丢失 |
| AWS | i3 / i3en | 实例停止后数据丢失 |
| 腾讯云 | IT5 系列 | 与实例生命周期绑定 |

> 使用本地 NVMe 实例时，**必须配置副本**以保障数据安全。

## 延伸阅读

- [RAID 配置]({{< relref "./raid.md" >}})
- [生产环境部署]({{< relref "/docs/deployment/install-guide/production-env.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})


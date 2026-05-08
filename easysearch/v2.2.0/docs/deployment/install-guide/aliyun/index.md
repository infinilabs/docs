---
title: "阿里云部署"
date: 0001-01-01
summary: "阿里云部署指南 #  本文介绍在阿里云 ECS 上部署 Easysearch 集群的推荐配置与实践。
推荐实例规格 #     节点角色 实例族 规格示例 说明     Master（专用） 通用型 g7 ecs.g7.xlarge (4C16G) 轻量计算，稳定即可   Data 存储增强型 i3 / 本地 SSD 型 ecs.i3.2xlarge (8C64G) 本地 NVMe SSD，高 IOPS   Data（云盘方案） 通用型 g7 ecs.g7.4xlarge (16C64G) 搭配 ESSD PL1/PL2   Coordinating 计算型 c7 ecs.c7.2xlarge (8C16G) 按查询并发酌情添加     生产环境至少 3 节点，跨可用区部署以实现高可用。
 存储选择 #     存储类型 IOPS 适用场景     本地 NVMe SSD (i 系列) 极高 高写入吞吐、大数据量   ESSD PL1 50,000 通用生产环境   ESSD PL2 100,000 高性能搜索场景   高效云盘 5,000 测试环境 / 冷数据    建议："
---


# 阿里云部署指南

本文介绍在阿里云 ECS 上部署 Easysearch 集群的推荐配置与实践。

## 推荐实例规格

| 节点角色 | 实例族 | 规格示例 | 说明 |
|----------|--------|----------|------|
| Master（专用） | 通用型 g7 | ecs.g7.xlarge (4C16G) | 轻量计算，稳定即可 |
| Data | 存储增强型 i3 / 本地 SSD 型 | ecs.i3.2xlarge (8C64G) | 本地 NVMe SSD，高 IOPS |
| Data（云盘方案） | 通用型 g7 | ecs.g7.4xlarge (16C64G) | 搭配 ESSD PL1/PL2 |
| Coordinating | 计算型 c7 | ecs.c7.2xlarge (8C16G) | 按查询并发酌情添加 |

> 生产环境至少 3 节点，跨可用区部署以实现高可用。

## 存储选择

| 存储类型 | IOPS | 适用场景 |
|----------|------|----------|
| 本地 NVMe SSD (i 系列) | 极高 | 高写入吞吐、大数据量 |
| ESSD PL1 | 50,000 | 通用生产环境 |
| ESSD PL2 | 100,000 | 高性能搜索场景 |
| 高效云盘 | 5,000 | 测试环境 / 冷数据 |

建议：

- 数据盘单独挂载到 `/data`，不要与系统盘混用
- 使用 ext4 或 xfs 文件系统
- 关闭 atime：`mount -o noatime /dev/vdb /data`

## 网络配置

### VPC 与安全组

```
VPC CIDR: 10.0.0.0/16
子网：
  - 10.0.1.0/24（可用区 A）
  - 10.0.2.0/24（可用区 B）
  - 10.0.3.0/24（可用区 C）
```

安全组规则（最小化）：

| 方向 | 端口 | 来源 | 说明 |
|------|------|------|------|
| 入方向 | 9200 | 应用服务器安全组 | REST API |
| 入方向 | 9300 | Easysearch 安全组 | 节点间通信 |
| 入方向 | 22 | 运维跳板机 | SSH 管理 |

> 不要将 9200 / 9300 端口暴露到公网。

### 跨可用区部署

将 3 个节点分别部署在不同可用区，配置 `node.attr.zone` 进行感知分配：

```yaml
# node-1 (可用区 A)
node.attr.zone: cn-hangzhou-a

# node-2 (可用区 B)
node.attr.zone: cn-hangzhou-b

# node-3 (可用区 C)
node.attr.zone: cn-hangzhou-c
```

配合分片分配感知：

```yaml
cluster.routing.allocation.awareness.attributes: zone
```

## 安装步骤

```bash
# 1. 系统调优（参考系统调优文档）
sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
ulimit -n 65535
swapoff -a

# 2. 创建用户
groupadd -g 602 easysearch
useradd -u 602 -g easysearch -m -d /home/easysearch easysearch

# 3. 一键安装
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch

# 4. 挂载数据盘
mkfs.ext4 /dev/vdb
mount -o noatime /dev/vdb /data
echo "/dev/vdb /data ext4 noatime 0 0" >> /etc/fstab

# 5. 配置 JDK
ln -s /usr/lib/jvm/java-17 /data/easysearch/jdk

# 6. 修改配置文件（见上方节点配置示例）

# 7. 初始化并启动
cd /data/easysearch
bin/initialize.sh -s
chown -R easysearch:easysearch /data/easysearch
su easysearch -c "/data/easysearch/bin/easysearch -d"
```

## 备份到 OSS

阿里云环境推荐使用 OSS 作为快照仓库：

```
PUT _snapshot/aliyun_oss
{
  "type": "fs",
  "settings": {
    "location": "/mnt/oss-backup/easysearch"
  }
}
```

> 可通过 ossfs 将 OSS bucket 挂载到本地路径，或使用 S3 兼容接口。

## 延伸阅读

- [生产环境部署]({{< relref "./production-env.md" >}})
- [分布式集群部署]({{< relref "./cluster.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})
- [硬件配置]({{< relref "/docs/deployment/config/hardware.md" >}})


---
title: "腾讯云部署"
date: 0001-01-01
summary: "腾讯云部署指南 #  本文介绍在腾讯云 CVM 上部署 Easysearch 集群的推荐配置与实践。
推荐实例规格 #     节点角色 实例族 规格示例 说明     Master（专用） 标准型 S6 S6.LARGE16 (4C16G) 轻量计算   Data 高 IO 型 IT5 IT5.4XLARGE64 (16C64G) 本地 NVMe SSD   Data（云盘方案） 内存型 M6 M6.4XLARGE64 (16C64G) 搭配增强型 SSD   Coordinating 计算型 C6 C6.2XLARGE16 (8C16G) 按查询并发酌情添加    存储选择 #     存储类型 IOPS 适用场景     本地 NVMe SSD (IT5) 极高 高性能搜索   增强型 SSD 50,000 通用生产环境   SSD 云硬盘 26,000 标准场景   高性能云硬盘 6,000 测试环境    网络配置 #  VPC 与安全组 #  VPC CIDR: 10."
---


# 腾讯云部署指南

本文介绍在腾讯云 CVM 上部署 Easysearch 集群的推荐配置与实践。

## 推荐实例规格

| 节点角色 | 实例族 | 规格示例 | 说明 |
|----------|--------|----------|------|
| Master（专用） | 标准型 S6 | S6.LARGE16 (4C16G) | 轻量计算 |
| Data | 高 IO 型 IT5 | IT5.4XLARGE64 (16C64G) | 本地 NVMe SSD |
| Data（云盘方案） | 内存型 M6 | M6.4XLARGE64 (16C64G) | 搭配增强型 SSD |
| Coordinating | 计算型 C6 | C6.2XLARGE16 (8C16G) | 按查询并发酌情添加 |

## 存储选择

| 存储类型 | IOPS | 适用场景 |
|----------|------|----------|
| 本地 NVMe SSD (IT5) | 极高 | 高性能搜索 |
| 增强型 SSD | 50,000 | 通用生产环境 |
| SSD 云硬盘 | 26,000 | 标准场景 |
| 高性能云硬盘 | 6,000 | 测试环境 |

## 网络配置

### VPC 与安全组

```
VPC CIDR: 10.0.0.0/16
子网：
  - 10.0.1.0/24 (可用区一)
  - 10.0.2.0/24 (可用区二)
  - 10.0.3.0/24 (可用区三)
```

安全组规则：

| 方向 | 端口 | 来源 | 说明 |
|------|------|------|------|
| 入站 | 9200 | 应用服务器安全组 | REST API |
| 入站 | 9300 | Easysearch 安全组 | 节点间通信 |
| 入站 | 22 | 堡垒机安全组 | SSH 管理 |

### 跨可用区部署

```yaml
# node-1 (可用区一)
node.attr.zone: ap-guangzhou-3

# node-2 (可用区二)
node.attr.zone: ap-guangzhou-4

# node-3 (可用区三)
node.attr.zone: ap-guangzhou-6
```

```yaml
cluster.routing.allocation.awareness.attributes: zone
```

## 安装步骤

```bash
# 1. 系统调优
sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
ulimit -n 65535
swapoff -a

# 2. 格式化并挂载数据盘
mkfs.ext4 /dev/vdb
mkdir -p /data
mount -o noatime /dev/vdb /data
echo "/dev/vdb /data ext4 noatime 0 0" >> /etc/fstab

# 3. 创建用户
groupadd -g 602 easysearch
useradd -u 602 -g easysearch -m easysearch

# 4. 安装
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch

# 5. 配置 JDK
ln -s /usr/lib/jvm/java-17 /data/easysearch/jdk

# 6. 修改 easysearch.yml

# 7. 初始化并启动
cd /data/easysearch
bin/initialize.sh -s
chown -R easysearch:easysearch /data/easysearch
su easysearch -c "/data/easysearch/bin/easysearch -d"
```

## 备份到 COS

腾讯云环境推荐使用对象存储 COS 作为快照仓库。可通过 cosfs 将 bucket 挂载到本地路径：

```bash
# 安装 cosfs
yum install -y cosfs

# 配置密钥
echo "your-bucket-name:your-secret-id:your-secret-key" > /etc/passwd-cosfs
chmod 600 /etc/passwd-cosfs

# 挂载
cosfs your-bucket-name /mnt/cos-backup -ourl=http://cos.ap-guangzhou.myqcloud.com
```

```
PUT _snapshot/cos_backup
{
  "type": "fs",
  "settings": {
    "location": "/mnt/cos-backup/easysearch"
  }
}
```

## 延伸阅读

- [生产环境部署]({{< relref "./production-env.md" >}})
- [分布式集群部署]({{< relref "./cluster.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})


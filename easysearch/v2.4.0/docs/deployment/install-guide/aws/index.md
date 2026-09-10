---
title: "AWS 部署"
date: 0001-01-01
summary: "AWS 部署指南 #  本文介绍在 AWS EC2 上部署 Easysearch 集群的推荐配置与实践。
推荐实例类型 #     节点角色 实例族 规格示例 说明     Master（专用） m6i / m7i m6i.xlarge (4C16G) 轻量计算   Data i3 / i3en i3.2xlarge (8C61G) 本地 NVMe SSD   Data（EBS 方案） r6i / r7i r6i.4xlarge (16C128G) 搭配 gp3/io2   Coordinating c6i / c7i c6i.2xlarge (8C16G) 按查询并发酌情添加     生产环境至少 3 节点，跨 AZ 部署。"
---


# AWS 部署指南

本文介绍在 AWS EC2 上部署 Easysearch 集群的推荐配置与实践。

## 推荐实例类型

| 节点角色 | 实例族 | 规格示例 | 说明 |
|----------|--------|----------|------|
| Master（专用） | m6i / m7i | m6i.xlarge (4C16G) | 轻量计算 |
| Data | i3 / i3en | i3.2xlarge (8C61G) | 本地 NVMe SSD |
| Data（EBS 方案） | r6i / r7i | r6i.4xlarge (16C128G) | 搭配 gp3/io2 |
| Coordinating | c6i / c7i | c6i.2xlarge (8C16G) | 按查询并发酌情添加 |

> 生产环境至少 3 节点，跨 AZ 部署。

## 存储选择

| 存储类型 | IOPS | 适用场景 |
|----------|------|----------|
| 实例存储 NVMe (i3) | 极高 | 高写入吞吐（注意：实例停止后数据丢失） |
| gp3 | 16,000 (可配置) | 通用生产，性价比高 |
| io2 | 64,000 | 低延迟搜索 |
| st1 | 500 | 冷数据 / 日志归档 |

gp3 配置建议：

```
卷大小: 按需
IOPS: 6000+（搜索场景建议 10000+）
吞吐量: 250+ MB/s
```

## 网络配置

### VPC 与安全组

```
VPC CIDR: 10.0.0.0/16
子网：
  - 10.0.1.0/24 (AZ-a)
  - 10.0.2.0/24 (AZ-b)
  - 10.0.3.0/24 (AZ-c)
```

安全组 Inbound Rules：

| Port | Source | Description |
|------|--------|-------------|
| 9200 | App Security Group | REST API |
| 9300 | ES Security Group | Node Transport |
| 22 | Bastion SG | SSH |

### 跨 AZ 部署

```yaml
# node-1 (AZ-a)
node.attr.zone: us-east-1a

# node-2 (AZ-b)
node.attr.zone: us-east-1b

# node-3 (AZ-c)
node.attr.zone: us-east-1c
```

```yaml
cluster.routing.allocation.awareness.attributes: zone
```

> 注意：跨 AZ 流量会产生额外费用，评估数据量与副本策略。

## 安装步骤

```bash
# 1. 系统调优
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo swapoff -a
ulimit -n 65535

# 2. 格式化并挂载数据盘（EBS gp3 示例）
sudo mkfs.xfs /dev/nvme1n1
sudo mkdir -p /data
sudo mount -o noatime /dev/nvme1n1 /data
echo "/dev/nvme1n1 /data xfs noatime 0 0" | sudo tee -a /etc/fstab

# 3. 创建用户
sudo groupadd -g 602 easysearch
sudo useradd -u 602 -g easysearch -m easysearch

# 4. 安装 Easysearch
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch

# 5. 配置 JVM
sudo ln -s /usr/lib/jvm/java-17 /data/easysearch/jdk

# 6. 修改 easysearch.yml（见上方节点配置）

# 7. 初始化并启动
cd /data/easysearch
sudo bin/initialize.sh -s
sudo chown -R easysearch:easysearch /data/easysearch
sudo -u easysearch bin/easysearch -d
```

## 备份到 S3

```
PUT _snapshot/s3_backup
{
  "type": "fs",
  "settings": {
    "location": "/mnt/s3-backup/easysearch"
  }
}
```

> 可通过 s3fs-fuse 挂载 S3 bucket，或使用 Easysearch 的 S3 快照仓库插件。详见 [S3 备份]({{< relref "/docs/deployment/install-guide/operator/s3_snapshot.md" >}})。

## 延伸阅读

- [生产环境部署]({{< relref "./production-env.md" >}})
- [分布式集群部署]({{< relref "./cluster.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})


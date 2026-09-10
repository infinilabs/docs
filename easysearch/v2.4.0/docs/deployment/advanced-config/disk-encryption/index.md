---
title: "磁盘加密"
date: 0001-01-01
summary: "磁盘加密配置指南 #  在合规要求较高的场景（金融、政务、医疗），可能需要对 Easysearch 数据所在磁盘进行静态加密（Encryption at Rest）。
加密方案对比 #     方案 透明性 性能影响 适用场景     LUKS (dm-crypt) 完全透明 5~15% Linux 原生，推荐   eCryptfs 文件级 15~30% 不推荐（性能差）   硬件自加密 (SED) 完全透明 ~0% 硬件支持时首选   云盘加密 完全透明 ~0% 云环境推荐    LUKS 全盘加密（Linux） #  配置步骤 #  # 1. 安装 cryptsetup yum install -y cryptsetup # CentOS/RHEL apt install -y cryptsetup # Ubuntu/Debian # 2."
---


# 磁盘加密配置指南

在合规要求较高的场景（金融、政务、医疗），可能需要对 Easysearch 数据所在磁盘进行静态加密（Encryption at Rest）。

## 加密方案对比

| 方案 | 透明性 | 性能影响 | 适用场景 |
|------|:------:|:--------:|----------|
| **LUKS (dm-crypt)** | 完全透明 | 5~15% | Linux 原生，推荐 |
| **eCryptfs** | 文件级 | 15~30% | 不推荐（性能差） |
| **硬件自加密 (SED)** | 完全透明 | ~0% | 硬件支持时首选 |
| **云盘加密** | 完全透明 | ~0% | 云环境推荐 |

## LUKS 全盘加密（Linux）

### 配置步骤

```bash
# 1. 安装 cryptsetup
yum install -y cryptsetup  # CentOS/RHEL
apt install -y cryptsetup  # Ubuntu/Debian

# 2. 加密磁盘（会清除数据！）
cryptsetup luksFormat /dev/nvme0n1
# 输入加密密码

# 3. 打开加密卷
cryptsetup open /dev/nvme0n1 es-data
# 创建 /dev/mapper/es-data

# 4. 格式化并挂载
mkfs.xfs -f /dev/mapper/es-data
mkdir -p /data/easysearch
mount -o noatime /dev/mapper/es-data /data/easysearch

# 5. 配置开机自动解锁
# 方式一：密钥文件（适合服务器）
dd if=/dev/urandom of=/root/.luks-key bs=256 count=1
chmod 400 /root/.luks-key
cryptsetup luksAddKey /dev/nvme0n1 /root/.luks-key

# /etc/crypttab
echo "es-data /dev/nvme0n1 /root/.luks-key luks" >> /etc/crypttab

# /etc/fstab
echo "/dev/mapper/es-data /data/easysearch xfs noatime 0 0" >> /etc/fstab
```

### 性能优化

```bash
# 使用 AES-NI 硬件加速（现代 CPU 均支持）
cryptsetup open --type luks --allow-discards /dev/nvme0n1 es-data

# 验证 AES-NI 是否启用
grep -o aes /proc/cpuinfo | head -1
# 输出 "aes" 表示支持

# 选择高效的加密参数
cryptsetup luksFormat --cipher aes-xts-plain64 --key-size 256 --hash sha256 /dev/nvme0n1
```

## 云环境加密

### 阿里云

```
创建云盘时勾选「加密」，选择 KMS 密钥
# 或通过 API：
aliyun ecs CreateDisk --Encrypted true --KMSKeyId <key-id>
```

### AWS

```
创建 EBS 卷时启用加密：
aws ec2 create-volume --encrypted --kms-key-id <key-id> ...
```

### 腾讯云

```
创建云硬盘时勾选「加密」
```

> 云盘加密对 Easysearch 完全透明，无需额外配置，也无明显性能损耗。

## 密钥管理建议

| 建议 | 说明 |
|------|------|
| 使用 KMS | 云环境使用云厂商的 KMS 管理密钥 |
| 密钥轮换 | 定期轮换加密密钥 |
| 备份密钥 | LUKS header 备份，防止密钥丢失导致数据不可恢复 |
| 分离存放 | 密钥不要存放在加密磁盘本身 |

```bash
# 备份 LUKS header（重要！）
cryptsetup luksHeaderBackup /dev/nvme0n1 --header-backup-file /backup/luks-header.bak
```

## 延伸阅读

- [TLS 安全配置]({{< relref "./tls-secure.md" >}})
- [国密配置]({{< relref "./guomi.md" >}})
- [安全模块总览]({{< relref "/docs/operations/security/" >}})


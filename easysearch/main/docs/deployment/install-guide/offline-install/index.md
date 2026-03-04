---
title: "离线安装"
date: 0001-01-01
summary: "离线安装 Easysearch #  本文档介绍如何在没有网络连接的环境中安装 Easysearch。离线安装常见于内网、政务、金融等对网络隔离有严格要求的场景。
准备工作（在有网络的环境中） #  在可联网的机器上提前下载所有需要的安装包：
必需文件 #     文件 用途 下载地址     Easysearch Bundle 包 包含 Easysearch + 内置 JDK  Linux AMD64   插件包（按需） IK 分词、Pinyin、KNN 等  插件列表     Bundle 包内置了 JDK，是离线安装的最简方式，无需单独准备 JDK。
 可选文件 #     文件 用途     INFINI Console 安装包 集群管理和监控   INFINI Gateway 安装包 查询代理和网关    Linux 环境离线安装 #  步骤 1：系统调优 #  在安装 Easysearch 之前，先完成操作系统调优（此步骤不需要网络）："
---


# 离线安装 Easysearch

本文档介绍如何在没有网络连接的环境中安装 Easysearch。离线安装常见于内网、政务、金融等对网络隔离有严格要求的场景。

## 准备工作（在有网络的环境中）

在可联网的机器上提前下载所有需要的安装包：

### 必需文件

| 文件 | 用途 | 下载地址 |
|------|------|----------|
| Easysearch Bundle 包 | 包含 Easysearch + 内置 JDK | [Linux AMD64](https://release.infinilabs.com/easysearch/stable/) |
| 插件包（按需） | IK 分词、Pinyin、KNN 等 | [插件列表](https://release.infinilabs.com/easysearch/stable/plugins/) |

> Bundle 包内置了 JDK，是离线安装的最简方式，无需单独准备 JDK。

### 可选文件

| 文件 | 用途 |
|------|------|
| INFINI Console 安装包 | 集群管理和监控 |
| INFINI Gateway 安装包 | 查询代理和网关 |

## Linux 环境离线安装

### 步骤 1：系统调优

在安装 Easysearch 之前，先完成操作系统调优（此步骤不需要网络）：

```bash
# 文件描述符
sudo tee /etc/security/limits.d/21-infini.conf <<-'EOF'
*    soft    nofile    1048576
*    hard    nofile    1048576
*    soft    memlock   unlimited
*    hard    memlock   unlimited
EOF

# 内核参数
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

> 完整调优参数参见 [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})。

### 步骤 2：创建用户

```bash
groupadd -g 602 easysearch
useradd -u 602 -g easysearch -m -d /home/easysearch -c 'easysearch' -s /bin/bash easysearch
```

### 步骤 3：解压安装

将预先下载的 Bundle 包传输到目标机器（通过 U 盘、SCP 等方式），然后解压安装：

```bash
# 创建安装目录
mkdir -p /data/easysearch

# 解压 bundle 包到安装目录
tar -zxf easysearch-*-linux-amd64-bundle.tar.gz -C /data/easysearch

# 初始化（生成证书和 admin 密码）
cd /data/easysearch && bin/initialize.sh -s

# 调整目录权限
chown -R easysearch:easysearch /data/easysearch
```

> 初始化完成后，admin 密码会在终端输出中显示，请务必记录。

### 步骤 4：安装插件（可选）

如需使用分词等插件，将提前下载的插件包传输到目标机器后安装：

```bash
# 切换到 easysearch 用户
su - easysearch

# 离线安装插件（以 analysis-ik 为例）
cd /data/easysearch
bin/easysearch-plugin install file:///tmp/analysis-ik-*.zip

# 查看已安装的插件
bin/easysearch-plugin list
```

### 步骤 5：启动并验证

```bash
# 启动 Easysearch
su easysearch -c "/data/easysearch/bin/easysearch -d -p pid"

# 验证（使用初始化时终端输出的密码）
curl -ku admin:YOUR_PASSWORD https://localhost:9200

# 停止 Easysearch
kill $(cat /data/easysearch/pid)
```

### 步骤 6：配置为系统服务（推荐）

```bash
# 创建 systemd 服务文件
sudo tee /usr/lib/systemd/system/easysearch.service <<-'EOF'
[Unit]
Description=Easysearch
After=network.target

[Service]
Type=forking
User=easysearch
ExecStart=/data/easysearch/bin/easysearch -d -p pid
PIDFile=/data/easysearch/pid
LimitNOFILE=65536
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable easysearch
sudo systemctl start easysearch
```

## Windows 环境离线安装

### 准备文件

在有网络的环境中下载：

- [Easysearch Windows 版](https://release.infinilabs.com/easysearch/stable/)
- [JDK 17 Windows 版](https://cdn.azul.com/zulu/bin/zulu17.54.21-ca-jre17.0.13-win_x64.zip)

### 安装步骤

1. 解压 Easysearch 到目标目录（如 `D:\easysearch`）
2. 解压 JDK 到 Easysearch 目录下，重命名为 `jdk`
3. 修改配置文件 `config/easysearch.yml`：

```yaml
# 如果无法在 Windows 环境生成证书，可临时禁用安全模块
security.enabled: false
```

4. 运行：

```bat
bin\easysearch.bat
```

> 如需启用安全功能，可在 Linux 环境提前生成证书，然后复制到 Windows 的 `config/` 目录。

## 离线升级

升级步骤与安装类似，使用[滚动重启]({{< relref "./production-env.md#滚动重启" >}})方式：

1. 下载新版本 Bundle 包并传输到目标机器
2. 按照滚动重启流程，逐节点停止、替换、启动
3. 注意保留 `config/`、`data/`、`logs/` 目录

## 延伸阅读

- [Linux 部署]({{< relref "./linux.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})
- [生产环境部署]({{< relref "./production-env.md" >}})
- [分布式集群部署]({{< relref "./cluster.md" >}})


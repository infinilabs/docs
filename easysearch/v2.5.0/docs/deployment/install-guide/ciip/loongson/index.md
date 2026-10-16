---
title: "龙芯平台安装"
date: 0001-01-01
summary: "龙芯平台安装 #  龙芯平台介绍 #  龙芯平台基于自主指令集架构（LoongArch），完全自主研发，不依赖国外技术，广泛应用于信创桌面、服务器及工控领域，强调安全可控与生态自主。
龙芯平台安装参考 #  目前，Easysearch 已支持在龙芯芯片的国产操作系统上运行，联网环境建议使用一键安装脚本进行安装，离线环境建议下载 Bundle 包进行安装，分布式集群安装请参考分布式集群安装。
 前提条件：已参照系统调优进行了系统优化，同时为 Easysearch 创建了专用的用户。
 JDK 25 下载与选择 #  Easysearch 2.5.0 在 LoongArch 平台推荐使用 JDK 25。官网下载目录提供两种构建：
   构建 下载 使用方式     默认包 loongson25.5.22-fx-jdk25.0.4_7-linux-loongarch64.tar.gz 初始化脚本默认下载   glibc2.34 包 loongson25.5.22-fx-jdk25.0.4_7-linux-loongarch64-glibc2.34.tar.gz 确认系统满足该构建的 glibc 和内核要求后手动安装    已有可用 jdk/ 或 JAVA_HOME 时，initialize.sh 会复用它；否则下载默认包。 initialize-cluster.sh 也使用默认包，支持本地 LoongArch 伪分布式部署及交互选择 LoongArch 目标架构。 如需使用 glibc2.34 构建，请先将其解压到安装目录的 jdk/，再运行初始化脚本。 已有 JDK 21 且暂不方便升级的客户环境仍可兼容运行；两种运行时都必须保留安全 agent 和 -Djava."
---


# 龙芯平台安装

## 龙芯平台介绍

龙芯平台基于自主指令集架构（LoongArch），完全自主研发，不依赖国外技术，广泛应用于信创桌面、服务器及工控领域，强调安全可控与生态自主。

### 龙芯平台安装参考

目前，Easysearch 已支持在龙芯芯片的国产操作系统上运行，联网环境建议使用一键安装脚本进行安装，离线环境建议下载 [Bundle 包](https://release.infinilabs.com/easysearch/stable/bundle/)进行安装，分布式集群安装请参考[分布式集群安装](../cluster.md)。

> 前提条件：已参照[系统调优]({{< relref "/docs/deployment/config/settings" >}})进行了系统优化，同时为 Easysearch 创建了专用的用户。

### JDK 25 下载与选择

Easysearch 2.5.0 在 LoongArch 平台推荐使用 JDK 25。官网下载目录提供两种构建：

| 构建 | 下载 | 使用方式 |
|------|------|----------|
| 默认包 | [loongson25.5.22-fx-jdk25.0.4_7-linux-loongarch64.tar.gz](https://release.infinilabs.com/easysearch/jdk/25/loongson25.5.22-fx-jdk25.0.4_7-linux-loongarch64.tar.gz) | 初始化脚本默认下载 |
| glibc2.34 包 | [loongson25.5.22-fx-jdk25.0.4_7-linux-loongarch64-glibc2.34.tar.gz](https://release.infinilabs.com/easysearch/jdk/25/loongson25.5.22-fx-jdk25.0.4_7-linux-loongarch64-glibc2.34.tar.gz) | 确认系统满足该构建的 glibc 和内核要求后手动安装 |

已有可用 `jdk/` 或 `JAVA_HOME` 时，`initialize.sh` 会复用它；否则下载默认包。
`initialize-cluster.sh` 也使用默认包，支持本地 LoongArch 伪分布式部署及交互选择 LoongArch 目标架构。
如需使用 `glibc2.34` 构建，请先将其解压到安装目录的 `jdk/`，再运行初始化脚本。
已有 JDK 21 且暂不方便升级的客户环境仍可兼容运行；两种运行时都必须保留安全 agent 和 `-Djava.security.manager=disallow`。

### 初始化系统参数及用户命令参考

```bash
# 调整内核配置
echo "vm.max_map_count=262144" >> /etc/sysctl.conf && sysctl -p
# 增加用户组与用户
groupadd -r easysearch && useradd -r -g easysearch -d /home/easysearch -s /sbin/nologin -c "Easysearch Service Account" easysearch
```

### 安装命令参考

```bash
# 创建数据目录
mkdir -p /data/easysearch
# 下载最新版本的 Easysearch 并安装
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch -d /data/easysearch
# 进入 Easysearch 目录
cd /data/easysearch
# 初始化 Easysearch
bin/initialize.sh -s
# 调整目录权限
chown -R easysearch:easysearch /data/easysearch
# 启动 Easysearch
runuser -u easysearch -- /data/easysearch/bin/easysearch -d -p /data/easysearch/easysearch.pid
```

> 注意：初始化过程中会生成随机密码，并不会保存到日志文件中，只会在终端显示一次，请妥善保存。如果忘记 `admin` 密码，可以使用  `bin/reset_admin_password.sh` 进行重置。


历史已验证系统环境（不代表上述 JDK 25 新包已完成运行验证）：
- Loongson-3C5000L/Loongnix-Server Linux release 8.4.1

如果您在其他龙芯平台的国产操作系统上安装遇到问题，欢迎通过[提交工单](https://www.infinilabs.cn/company/contact/)与我们联系。


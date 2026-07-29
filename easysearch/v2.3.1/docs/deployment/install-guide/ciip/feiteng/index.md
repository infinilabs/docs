---
title: "飞腾平台安装"
date: 0001-01-01
summary: "飞腾平台安装 #  飞腾平台介绍 #  飞腾平台基于 ARM 架构，由飞腾公司自主研发，提供高性能、低功耗的 CPU 产品，广泛应用于信创桌面、服务器及嵌入式领域，全面适配统信 UOS、麒麟等国产操作系统，支撑党政、金融、电信等行业国产化替代需求。
飞腾平台安装参考 #  目前，Easysearch 已支持在飞腾芯片的国产操作系统上运行，联网环境建议使用一键安装脚本进行安装，离线环境建议下载 Bundle 包进行安装，分布式集群安装请参考 分布式集群安装。
 前提条件：已参照 系统调优进行了系统优化，同时为 Easysearch 创建了专用的用户。
 初始化系统参数及用户命令参考 #  # 调整内核配置 echo &#34;vm.max_map_count=262144&#34; &gt;&gt; /etc/sysctl.conf &amp;&amp; sysctl -p # 增加用户组与用户 groupadd -r easysearch &amp;&amp; useradd -r -g easysearch -d /home/easysearch -s /sbin/nologin -c &#34;Easysearch Service Account&#34; easysearch 安装命令参考 #  # 创建数据目录 mkdir -p /opt/easysearch # 下载最新版本的 Easysearch 并安装 curl -sSL http://get."
---


# 飞腾平台安装

## 飞腾平台介绍

飞腾平台基于 ARM 架构，由飞腾公司自主研发，提供高性能、低功耗的 CPU 产品，广泛应用于信创桌面、服务器及嵌入式领域，全面适配统信 UOS、麒麟等国产操作系统，支撑党政、金融、电信等行业国产化替代需求。

### 飞腾平台安装参考

目前，Easysearch 已支持在飞腾芯片的国产操作系统上运行，联网环境建议使用一键安装脚本进行安装，离线环境建议下载 [Bundle 包](https://release.infinilabs.com/easysearch/stable/bundle/)进行安装，分布式集群安装请参考[分布式集群安装](../cluster.md)。


> 前提条件：已参照[系统调优]({{< relref "/docs/deployment/config/settings" >}})进行了系统优化，同时为 Easysearch 创建了专用的用户。

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
mkdir -p /opt/easysearch
# 下载最新版本的 Easysearch 并安装
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch -d /opt/easysearch
# 进入 Easysearch 目录
cd /opt/easysearch
# 初始化 Easysearch
bin/initialize.sh -s
# 调整目录权限
chown -R easysearch:easysearch /opt/easysearch
# 启动 Easysearch
runuser -u easysearch -- /opt/easysearch/bin/easysearch -d -p /opt/easysearch/easysearch.pid
```

> 注意：初始化过程中会生成随机密码，并不会保存到日志文件中，只会在终端显示一次，请妥善保存。如果忘记 `admin` 密码，可以使用  `bin/reset_admin_password.sh` 进行重置。

已验证系统环境：
- 腾云 S2500/银河麒麟高级服务器操作系统V10 SP3

如果您在其他飞腾平台的国产操作系统上安装遇到问题，欢迎通过[提交工单](https://www.infinilabs.cn/company/contact/)与我们联系。


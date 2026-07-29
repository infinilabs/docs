---
title: "部署手册"
date: 0001-01-01
description: "各平台部署与配置指南。"
summary: "部署手册 #  各平台的详细部署说明、参数配置与高阶调优指南。
章节导航 #  安装指南 #  各平台安装方式：
 平台安装： Linux / MacOS / Windows 容器与编排： Docker / Docker Compose / Helm Chart / Operator 环境与场景： 测试环境 / 生产环境 / 离线安装 / IPv6 集群部署： 分布式集群 / 一键安装 Console + Easysearch 云平台部署： 阿里云 / AWS / 腾讯云 信创平台：龙芯、兆芯、鲲鹏、飞腾、海光、麒麟、统信等  参数配置 #    节点配置：easysearch.yml、jvm.options、log4j2.properties 详解  系统调优：Linux 内核参数、文件描述符、虚拟内存  硬件配置：CPU / 内存 / 磁盘 / 网络选型建议  集群配置：集群设置 API、动态配置项参考  高阶配置与调优 #    NUMA 配置：多路 CPU 服务器的内存亲和性优化  NVMe 配置：高性能 SSD 格式化、挂载与调度器设置  RAID 配置：JBOD 多路径 vs."
---


# 部署手册

各平台的详细部署说明、参数配置与高阶调优指南。

## 章节导航

### [安装指南]({{< relref "./install-guide/" >}})

各平台安装方式：

- **平台安装**：[Linux]({{< relref "./install-guide/linux.md" >}}) / [MacOS]({{< relref "./install-guide/macos.md" >}}) / [Windows]({{< relref "./install-guide/windows.md" >}})
- **容器与编排**：[Docker]({{< relref "./install-guide/docker.md" >}}) / [Docker Compose]({{< relref "./install-guide/docker-compose.md" >}}) / [Helm Chart]({{< relref "./install-guide/helm.md" >}}) / [Operator]({{< relref "./install-guide/operator/" >}})
- **环境与场景**：[测试环境]({{< relref "./install-guide/test-env.md" >}}) / [生产环境]({{< relref "./install-guide/production-env.md" >}}) / [离线安装]({{< relref "./install-guide/offline-install.md" >}}) / [IPv6]({{< relref "./install-guide/ipv6.md" >}})
- **集群部署**：[分布式集群]({{< relref "./install-guide/cluster.md" >}}) / [一键安装 Console + Easysearch]({{< relref "./install-guide/console.md" >}})
- **云平台部署**：[阿里云]({{< relref "./install-guide/aliyun.md" >}}) / [AWS]({{< relref "./install-guide/aws.md" >}}) / [腾讯云]({{< relref "./install-guide/tencent-cloud.md" >}})
- **信创平台**：龙芯、兆芯、鲲鹏、飞腾、海光、麒麟、统信等

### [参数配置]({{< relref "./config/" >}})

- [节点配置]({{< relref "./config/node-settings/" >}})：`easysearch.yml`、`jvm.options`、`log4j2.properties` 详解
- [系统调优]({{< relref "./config/settings.md" >}})：Linux 内核参数、文件描述符、虚拟内存
- [硬件配置]({{< relref "./config/hardware.md" >}})：CPU / 内存 / 磁盘 / 网络选型建议
- [集群配置]({{< relref "./config/configuration.md" >}})：集群设置 API、动态配置项参考

### [高阶配置与调优]({{< relref "./advanced-config/" >}})

- [NUMA 配置]({{< relref "./advanced-config/numa.md" >}})：多路 CPU 服务器的内存亲和性优化
- [NVMe 配置]({{< relref "./advanced-config/nvme.md" >}})：高性能 SSD 格式化、挂载与调度器设置
- [RAID 配置]({{< relref "./advanced-config/raid.md" >}})：JBOD 多路径 vs. RAID 方案对比
- [磁盘加密]({{< relref "./advanced-config/disk-encryption.md" >}})：LUKS、云盘加密与密钥管理
- [TLS 安全配置]({{< relref "./advanced-config/tls-secure.md" >}})：证书管理、TLS 版本与密码套件
- [国密配置]({{< relref "./advanced-config/guomi.md" >}})：SM2/SM3/SM4 国密算法 TLS 配置

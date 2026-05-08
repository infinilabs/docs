---
title: "如何安装"
date: 0001-01-01
description: "选择适合你环境的安装方式，快速部署 Easysearch。"
summary: "如何安装 #  根据你的运行环境选择合适的安装方式。
推荐方式 #  # 一键安装（Linux） curl -sSL http://get.infini.cloud | bash -s -- -p easysearch cd /data/easysearch &amp;&amp; bin/initialize.sh -s chown -R easysearch:easysearch /data/easysearch su easysearch -c &#34;/data/easysearch/bin/easysearch -d&#34; 各平台安装 #     平台 文档     Linux  Linux 安装   Windows  Windows 安装   MacOS  MacOS 安装   Docker  Docker 部署   Docker Compose  Docker Compose 集群   Kubernetes  Helm 部署    环境类型 #     类型 文档     测试环境  测试环境部署：最低资源，快速验证   生产环境  生产环境部署：高可用配置   分布式集群  集群部署：多节点组网    云平台 #     平台 文档     阿里云  阿里云部署   AWS  AWS 部署   腾讯云  腾讯云部署    信创平台 #  支持龙芯、兆芯、鲲鹏、飞腾、麒麟、统信等国产平台，详见 安装指南 - 信创平台。"
---


# 如何安装

根据你的运行环境选择合适的安装方式。

## 推荐方式

```bash
# 一键安装（Linux）
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch
cd /data/easysearch && bin/initialize.sh -s
chown -R easysearch:easysearch /data/easysearch
su easysearch -c "/data/easysearch/bin/easysearch -d"
```

## 各平台安装

| 平台 | 文档 |
|------|------|
| **Linux** | [Linux 安装]({{< relref "/docs/deployment/install-guide/linux.md" >}}) |
| **Windows** | [Windows 安装]({{< relref "/docs/deployment/install-guide/windows.md" >}}) |
| **MacOS** | [MacOS 安装]({{< relref "/docs/deployment/install-guide/macos.md" >}}) |
| **Docker** | [Docker 部署]({{< relref "/docs/deployment/install-guide/docker.md" >}}) |
| **Docker Compose** | [Docker Compose 集群]({{< relref "/docs/deployment/install-guide/docker-compose.md" >}}) |
| **Kubernetes** | [Helm 部署]({{< relref "/docs/deployment/install-guide/helm.md" >}}) |

## 环境类型

| 类型 | 文档 |
|------|------|
| **测试环境** | [测试环境部署]({{< relref "/docs/deployment/install-guide/test-env.md" >}})：最低资源，快速验证 |
| **生产环境** | [生产环境部署]({{< relref "/docs/deployment/install-guide/production-env.md" >}})：高可用配置 |
| **分布式集群** | [集群部署]({{< relref "/docs/deployment/install-guide/cluster.md" >}})：多节点组网 |

## 云平台

| 平台 | 文档 |
|------|------|
| **阿里云** | [阿里云部署]({{< relref "/docs/deployment/install-guide/aliyun.md" >}}) |
| **AWS** | [AWS 部署]({{< relref "/docs/deployment/install-guide/aws.md" >}}) |
| **腾讯云** | [腾讯云部署]({{< relref "/docs/deployment/install-guide/tencent-cloud.md" >}}) |

## 信创平台

支持龙芯、兆芯、鲲鹏、飞腾、麒麟、统信等国产平台，详见 [安装指南 - 信创平台]({{< relref "/docs/deployment/install-guide/" >}})。

> 更完整的部署配置（系统调优、JVM、插件管理等）请参阅 [部署手册]({{< relref "/docs/deployment/" >}})。

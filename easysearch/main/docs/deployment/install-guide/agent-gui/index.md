---
title: "图形化部署"
date: 0001-01-01
summary: "通过 Agent 图形界面部署 Easysearch #  Agent 提供图形界面，用于安装 Easysearch、组建集群和管理节点。
在每台计划部署 Easysearch 节点的机器上运行 Agent。先选择一台机器，以生产模式创建集群的第一个节点。创建完成后，在其他机器的 Agent 中选择「加入现有集群」，使用已有集群的准入凭证添加节点，并按需选择节点角色。
重复加入操作，即可扩展集群。
1. 在各台机器上运行 Agent #  准备多台物理机或虚拟机，在每台机器上安装并运行一个 Agent 进程。安装和启动方式见安装 INFINI Agent。
浏览器访问对应机器的 Agent 页面，默认端口为 23000。例如，机器地址为 192.168.1.10 时，访问 http://192.168.1.10:23000。
使用该 Agent 的登录 Token 登录。Token 的获取方式见认证。每台机器的 Agent 分别管理本机的 Easysearch 服务。
部署前，确认机器之间网络互通。Easysearch 的 Transport 端口用于节点间通信，默认 9300；HTTP 端口用于访问集群 API，默认 9200。监听地址需使用其他机器可访问的地址。
2. 准备证书 #  生产模式启用安全功能和 TLS。可以让 Agent 自动生成证书，也可以自行准备证书，在配置页面选择「上传现有证书」。
HTTP 和 Transport 可以共用一套证书，也可以分别配置。自行准备证书时，使用同一个 Transport CA 为各节点签发证书，确保节点间相互信任。HTTP 证书按实际访问的域名或 IP 地址准备。
如果各节点使用独立证书，加入节点时选择自定义证书配置，并确保已有节点和新节点的节点证书 DN 白名单相互匹配。证书准备方式见 TLS 安全配置。"
---


# 通过 Agent 图形界面部署 Easysearch

Agent 提供图形界面，用于安装 Easysearch、组建集群和管理节点。

在每台计划部署 Easysearch 节点的机器上运行 Agent。先选择一台机器，以生产模式创建集群的第一个节点。创建完成后，在其他机器的 Agent 中选择「加入现有集群」，使用已有集群的准入凭证添加节点，并按需选择节点角色。

重复加入操作，即可扩展集群。

{{% load-img "/img/deployment/install-guide/agent-gui/agent-gui-deploy-easysearch.png" %}}

## 1. 在各台机器上运行 Agent

准备多台物理机或虚拟机，在每台机器上安装并运行一个 Agent 进程。安装和启动方式见[安装 INFINI Agent](https://docs.infinilabs.com/agent/main/zh/docs/getting-started/install/)。

浏览器访问对应机器的 Agent 页面，默认端口为 `23000`。例如，机器地址为 `192.168.1.10` 时，访问 `http://192.168.1.10:23000`。

使用该 Agent 的登录 Token 登录。Token 的获取方式见[认证](https://docs.infinilabs.com/agent/main/zh/docs/getting-started/api-auth/)。每台机器的 Agent 分别管理本机的 Easysearch 服务。

部署前，确认机器之间网络互通。Easysearch 的 Transport 端口用于节点间通信，默认 `9300`；HTTP 端口用于访问集群 API，默认 `9200`。监听地址需使用其他机器可访问的地址。

## 2. 准备证书

生产模式启用安全功能和 TLS。可以让 Agent 自动生成证书，也可以自行准备证书，在配置页面选择「上传现有证书」。

HTTP 和 Transport 可以共用一套证书，也可以分别配置。自行准备证书时，使用同一个 Transport CA 为各节点签发证书，确保节点间相互信任。HTTP 证书按实际访问的域名或 IP 地址准备。

如果各节点使用独立证书，加入节点时选择自定义证书配置，并确保已有节点和新节点的节点证书 DN 白名单相互匹配。证书准备方式见 [TLS 安全配置]({{< relref "/docs/deployment/advanced-config/tls-secure.md" >}})。

## 3. 创建第一个节点

选择一台机器，进入其 Agent 的服务管理页面，点击「创建集群」，选择「生产模式 | 新集群」。

按页面提示选择 Easysearch 和 JDK 版本，配置网络、管理员密码、证书、JVM 内存等参数。在「集群设置」中填写集群名称、节点名称，并按部署规划选择节点角色。

如需在 Agent 启动时自动启动该节点，开启「随 Agent 启动」。配置完成后，点击「开始安装」，等待节点创建成功。

具体操作见[创建生产模式集群]({{< relref "/docs/management/service-management/create-prod-cluster.md" >}})。

## 4. 加入其他节点

从第一个节点的 Agent 服务管理页面进入集群概览，点击「获取集群准入凭证」并复制凭证。

打开另一台机器的 Agent，在服务管理页面点击「加入现有集群」，填写准入凭证并验证连接。验证通过后，配置节点名称、监听地址、端口、节点角色、JVM 内存和数据目录等参数。

证书可以复用集群共享证书，也可以使用自行准备的节点证书。配置完成后，点击「开始安装」，等待节点加入成功。

在其余机器上重复以上加入操作。完成后，在集群的节点列表中确认节点数量、名称和角色符合部署规划。

具体操作和准入凭证的获取方式见[加入已有集群]({{< relref "/docs/management/service-management/join-cluster.md" >}})。


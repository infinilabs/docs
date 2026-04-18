---
title: "图形化部署"
date: 0001-01-01
summary: "通过 Agent 图形界面部署 Easysearch #  Agent 支持图形化一键拉起新集群（开发/生产双模式），用户无需手动编辑任何配置文件，通过图形界面即可完成 Easysearch 集群的安装、配置和日常管理。
  安装 Agent
macOS/Linux 用户可以通过如下脚本进行快速安装：
curl -sSL http://get.infini.cloud | bash -s -- -p agent Windows 用户可以手动下载安装包进行安装，见 安装 INFINI Agent。
  运行 Agent
  Linux / macOS
cd agent-x.y.z-xxxx ./agent-x.y.z   Windows
cd agent-x.y.z-xxxx .\agent-x.y.z.exe     浏览器访问 Agent
Agent UI 默认监听在 http://127.0.0.1:9000，浏览器打开此地址。
  登录 Agent
见 服务管理登录
  创建 Easysearch 集群
图形化部署支持 2 种创建集群的方式，见:"
---


# 通过 Agent 图形界面部署 Easysearch

Agent 支持图形化一键拉起新集群（开发/生产双模式），用户无需手动编辑任何配置文件，通过图形界面即可完成 Easysearch 集群的安装、配置和日常管理。

{{% load-img "/img/deployment/install-guide/agent-gui/agent-gui-deploy-easysearch.png" %}}


1. 安装 Agent

   macOS/Linux 用户可以通过如下脚本进行快速安装：

   ```sh
   curl -sSL http://get.infini.cloud | bash -s -- -p agent
   ```

   Windows 用户可以手动下载安装包进行安装，见[安装 INFINI Agent](https://docs.infinilabs.com/agent/main/zh/docs/getting-started/install/)。

3. 运行 Agent

   * Linux / macOS

     ```sh
     cd agent-x.y.z-xxxx
     ./agent-x.y.z
     ```

   * Windows

     ```powershell
     cd agent-x.y.z-xxxx
     .\agent-x.y.z.exe
     ```

4. 浏览器访问 Agent

   Agent UI 默认监听在 `http://127.0.0.1:9000`，浏览器打开此地址。

5. 登录 Agent

   见[服务管理登录](/docs/management/service-management/login/)

6. 创建 Easysearch 集群

   图形化部署支持 2 种创建集群的方式，见:

   1. [创建开发模式集群](/docs/management/service-management/create-dev-cluster/)
   2. [创建生产模式集群](/docs/management/service-management/create-prod-cluster/)

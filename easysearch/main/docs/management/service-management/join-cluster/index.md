---
title: "加入已有集群"
date: 0001-01-01
description: "在服务管理平台中加入已有集群。"
summary: "加入已有集群 #  可以将当前节点加入到一个已有的 Easysearch 集群中。
操作步骤 #   进入服务管理页面：在 Agent UI 中进入「服务管理」页面，点击「加入现有集群」按钮，进入安装向导。  完成系统环境检测：系统会自动进行环境兼容性检测，确认所有检测项均通过后，点击「开始安装」。  配置集群连接信息：  输入「加入凭证」，点击「验证连接」。 验证通过后，系统将自动获取并展示集群信息。    配置节点信息：  填写节点名称、监听地址、HTTP 端口、Transport 端口，并选择节点角色。     配置 JVM 内存、数据/日志目录等参数，完成后点击「开始安装」。  等待下载与同步：系统将自动下载 Easysearch、JDK 并同步集群配置，等待进度条完成。  完成集群加入：安装完成后，页面将显示节点已成功加入集群，点击「打开 Easysearch」即可访问服务。  注意事项 #   加入集群前，请确保目标集群处于正常运行状态，且节点间网络互通。 节点的 HTTP 端口和 Transport 端口需为空闲端口，可通过「检测端口」功能进行校验。 节点角色配置需与集群规划一致，避免角色冲突影响集群可用性。  "
---


# 加入已有集群

可以将当前节点加入到一个已有的 Easysearch 集群中。

## 操作步骤

1. **进入服务管理页面**：在 Agent UI 中进入「服务管理」页面，点击「加入现有集群」按钮，进入安装向导。

{{% load-img "/img/management/service-management/join-cluster/image-1.png" %}}

2. **完成系统环境检测**：系统会自动进行环境兼容性检测，确认所有检测项均通过后，点击「开始安装」。

{{% load-img "/img/management/service-management/join-cluster/image-2.png" %}}

3. **配置集群连接信息**：
   - 输入「加入凭证」，点击「验证连接」。
   - 验证通过后，系统将自动获取并展示集群信息。

{{% load-img "/img/management/service-management/join-cluster/image-3.png" %}}

4. **配置节点信息**：
   - 填写节点名称、监听地址、HTTP 端口、Transport 端口，并选择节点角色。

{{% load-img "/img/management/service-management/join-cluster/image-4.png" %}}

- 配置 JVM 内存、数据/日志目录等参数，完成后点击「开始安装」。

{{% load-img "/img/management/service-management/join-cluster/image-5.png" %}}

5. **等待下载与同步**：系统将自动下载 Easysearch、JDK 并同步集群配置，等待进度条完成。

{{% load-img "/img/management/service-management/join-cluster/image-6.png" %}}

6. **完成集群加入**：安装完成后，页面将显示节点已成功加入集群，点击「打开 Easysearch」即可访问服务。

{{% load-img "/img/management/service-management/join-cluster/image-7.png" %}}

## 注意事项

- 加入集群前，请确保目标集群处于正常运行状态，且节点间网络互通。
- 节点的 HTTP 端口和 Transport 端口需为空闲端口，可通过「检测端口」功能进行校验。
- 节点角色配置需与集群规划一致，避免角色冲突影响集群可用性。


---
title: "节点详情的查看和管理"
date: 0001-01-01
description: "在 Easysearch UI 中查看和管理节点的详细信息。"
summary: "查看节点详情 #  通过节点详情页，可一站式查看节点的完整运行数据。
进入详情页面 #    进入节点列表页：在左侧导航栏点击「节点」，进入节点管理列表页面，可查看集群内所有节点的基础运行信息。
  进入节点详情页：点击目标节点的名称，即可进入该节点的详情页面。
  详情页按标签页组织：概览、分片、配置、密钥库、插件、日志。其中配置、密钥库、插件、日志由 Agent 提供，需要该节点为 Agent 创建的节点且已连接 Agent 才会显示。
查看节点概览 #  「概览」标签页展示节点的核心运行指标，包括 CPU 使用率、JVM 堆内存、文档数、删除文档数、可用存储、数据目录等信息，数据会定期自动刷新，可用于快速判断节点的健康与负载状况。
查看节点分片列表 #  「分片」标签页列出该节点承载的全部分片，可查看每个分片所属的索引、主/副本角色、状态、存储大小与文档数，便于排查分片在节点上的分布与磁盘占用问题。
查看节点配置 #   若此 Easysearch 节点为 Agent 创建的节点，连接 Agent 后才有此功能
 「配置」标签页用于查看和管理该节点的本地配置，包括版本信息、JVM 配置与日志配置（日志级别、滚动策略、单文件大小、保留文件数、输出格式、异步日志等）。修改保存后按提示重启节点生效。
查看节点密钥库 #   若此 Easysearch 节点为 Agent 创建的节点，连接 Agent 后才有此功能
 「密钥库」标签页用于管理该节点的安全配置（secure settings）。安全配置保存在节点本地的密钥库文件中。
列表展示当前密钥库中的全部配置项名称，支持按名称搜索。
新增、更新密钥配置 #  点击列表上方的「新增」按钮，填写配置项名称与配置项值即可创建:
编辑已有配置时名称不可修改，输入新值保存即完成覆盖更新。
删除密钥配置 #  点击目标配置项操作列的「···」菜单，选择「删除」并在确认框中确认。"
---


# 查看节点详情

通过节点详情页，可一站式查看节点的完整运行数据。

## 进入详情页面

1. **进入节点列表页**：在左侧导航栏点击「节点」，进入节点管理列表页面，可查看集群内所有节点的基础运行信息。

2. **进入节点详情页**：点击目标节点的名称，即可进入该节点的详情页面。

   {{% load-img "/img/management/nodes/view-node-details/image-1.png" %}}


详情页按标签页组织：概览、分片、配置、密钥库、插件、日志。其中配置、密钥库、插件、日志由 Agent 提供，需要该节点为 Agent 创建的节点且已连接 Agent 才会显示。

{{% load-img "/img/management/nodes/view-node-details/image-2.png" %}}

### 查看节点概览

「概览」标签页展示节点的核心运行指标，包括 CPU 使用率、JVM 堆内存、文档数、删除文档数、可用存储、数据目录等信息，数据会定期自动刷新，可用于快速判断节点的健康与负载状况。

{{% load-img "/img/management/nodes/view-node-details/image-3.png" %}}

### 查看节点分片列表

「分片」标签页列出该节点承载的全部分片，可查看每个分片所属的索引、主/副本角色、状态、存储大小与文档数，便于排查分片在节点上的分布与磁盘占用问题。

{{% load-img "/img/management/nodes/view-node-details/image-4.png" %}}

### 查看节点配置

> 若此 Easysearch 节点为 Agent 创建的节点，连接 Agent 后才有此功能

「配置」标签页用于查看和管理该节点的本地配置，包括版本信息、JVM 配置与日志配置（日志级别、滚动策略、单文件大小、保留文件数、输出格式、异步日志等）。修改保存后按提示重启节点生效。

{{% load-img "/img/management/nodes/view-node-details/image-5.png" %}}

### 查看节点密钥库

> 若此 Easysearch 节点为 Agent 创建的节点，连接 Agent 后才有此功能

「密钥库」标签页用于管理该节点的安全配置（secure settings）。安全配置保存在节点本地的密钥库文件中。

{{% load-img "/img/management/nodes/view-node-details/image-6.png" %}}

列表展示当前密钥库中的全部配置项名称，支持按名称搜索。

#### 新增、更新密钥配置

点击列表上方的「新增」按钮，填写配置项名称与配置项值即可创建:

{{% load-img "/img/management/nodes/view-node-details/image-7.png" %}}
{{% load-img "/img/management/nodes/view-node-details/image-8.png" %}}

编辑已有配置时名称不可修改，输入新值保存即完成覆盖更新。

{{% load-img "/img/management/nodes/view-node-details/image-9.png" %}}

#### 删除密钥配置

点击目标配置项操作列的「···」菜单，选择「删除」并在确认框中确认。

{{% load-img "/img/management/nodes/view-node-details/image-10.png" %}}

#### 重新加载使变更生效

任何编辑操作（新增、更新、删除）都不会自动生效，需要点击列表上方的「重新加载」按钮，让节点从密钥库文件重新加载安全配置后，变更才会生效。

{{% load-img "/img/management/nodes/view-node-details/image-11.png" %}}

### 查看节点插件列表

> 若此 Easysearch 节点为 Agent 创建的节点，连接 Agent 后可以进行插件的管理。若非 Agent 创建的节点或未连接 Agent，则只能查看已安装的插件列表。

「插件」标签页列出该节点已安装的全部插件，可查看插件名称、描述与版本，并支持上传本地插件或安装在线插件、卸载已有插件。

{{% load-img "/img/management/nodes/view-node-details/image-12.png" %}}

### 查看节点日志

> 若此 Easysearch 节点为 Agent 创建的节点，连接 Agent 后才有此功能

「日志」标签页可在线查看该节点的服务日志，支持按关键词过滤、按日志级别筛选与自动刷新，无需登录宿主机即可排查节点运行问题。

{{% load-img "/img/management/nodes/view-node-details/image-13.png" %}}


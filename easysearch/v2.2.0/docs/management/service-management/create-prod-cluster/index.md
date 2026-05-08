---
title: "创建生产模式集群"
date: 0001-01-01
description: "在服务管理平台中创建生产模式集群。"
summary: "创建生产模式集群 #  操作步骤 #   进入服务管理页面：登录 Agent UI 后，在服务管理页面，点击「安装 Easysearch」区域的「创建集群」按钮。  完成系统环境检测：进入安装向导，系统会自动检测操作系统、内存、JDK 等安装前置条件，确认所有项均通过后，点击「开始安装」。  选择集群模式：在安装模式选择页，点击「生产模式 | 新集群」卡片，进入生产模式配置流程。  配置基础环境信息：  选择 Easysearch 版本、JDK 版本。 填写网络配置：监听地址、HTTP 端口、Transport 端口，点击「检测端口」确保端口可用。 设置管理员密码，可使用「生成密码并复制」快速生成强密码。    完成高级配置项：  配置安全模式，可选择自动生成 CA 和节点证书或上传现有证书。    - 设置 JVM 内存，建议根据系统可用内存分配。 - 填写集群名称、节点名称，可按需开启自定义数据目录、日志目录。 - 勾选需要安装的插件（如 ai、analysis-ik 等）。  启动集群创建流程：配置完成后，点击页面底部的「开始安装」，系统将自动执行下载、解压、配置证书、启动服务等步骤。  确认集群创建结果：等待安装进度完成，页面提示「集群创建成功」后，可查看集群状态、节点信息，点击「打开 Easysearch」进入集群管理界面。  注意事项 #   生产模式集群对系统资源要求较高，建议在安装前确保系统内存、CPU 等资源满足推荐配置，避免因资源不足导致集群性能异常。 端口配置需确保未被其他进程占用，HTTP 端口默认 9200、Transport 端口默认 9300，如已被占用需手动修改。 管理员密码需妥善保存，若使用自动生成密码，需提前复制保存，避免后续无法登录集群。 插件安装需根据业务需求选择，安装过多不必要的插件可能影响集群启动速度和运行性能。  "
---


# 创建生产模式集群

## 操作步骤

1. **进入服务管理页面**：登录 Agent UI 后，在服务管理页面，点击「安装 Easysearch」区域的「创建集群」按钮。

{{% load-img "/img/management/service-management/create-prod-cluster/image-1.png" %}}

2. **完成系统环境检测**：进入安装向导，系统会自动检测操作系统、内存、JDK 等安装前置条件，确认所有项均通过后，点击「开始安装」。

{{% load-img "/img/management/service-management/create-prod-cluster/image-2.png" %}}

3. **选择集群模式**：在安装模式选择页，点击「生产模式 | 新集群」卡片，进入生产模式配置流程。

{{% load-img "/img/management/service-management/create-prod-cluster/image-3.png" %}}

4. **配置基础环境信息**：
   - 选择 Easysearch 版本、JDK 版本。
   - 填写网络配置：监听地址、HTTP 端口、Transport 端口，点击「检测端口」确保端口可用。
   - 设置管理员密码，可使用「生成密码并复制」快速生成强密码。

{{% load-img "/img/management/service-management/create-prod-cluster/image-4.png" %}}

5. **完成高级配置项**：
   - 配置安全模式，可选择自动生成 CA 和节点证书或上传现有证书。

{{% load-img "/img/management/service-management/create-prod-cluster/image-5.png" %}}

    - 设置 JVM 内存，建议根据系统可用内存分配。
    - 填写集群名称、节点名称，可按需开启自定义数据目录、日志目录。
    - 勾选需要安装的插件（如 ai、analysis-ik 等）。

{{% load-img "/img/management/service-management/create-prod-cluster/image-6.png" %}}

6. **启动集群创建流程**：配置完成后，点击页面底部的「开始安装」，系统将自动执行下载、解压、配置证书、启动服务等步骤。

{{% load-img "/img/management/service-management/create-prod-cluster/image-7.png" %}}

7. **确认集群创建结果**：等待安装进度完成，页面提示「集群创建成功」后，可查看集群状态、节点信息，点击「打开 Easysearch」进入集群管理界面。

{{% load-img "/img/management/service-management/create-prod-cluster/image-8.png" %}}

## 注意事项

- 生产模式集群对系统资源要求较高，建议在安装前确保系统内存、CPU 等资源满足推荐配置，避免因资源不足导致集群性能异常。
- 端口配置需确保未被其他进程占用，HTTP 端口默认 9200、Transport 端口默认 9300，如已被占用需手动修改。
- 管理员密码需妥善保存，若使用自动生成密码，需提前复制保存，避免后续无法登录集群。
- 插件安装需根据业务需求选择，安装过多不必要的插件可能影响集群启动速度和运行性能。


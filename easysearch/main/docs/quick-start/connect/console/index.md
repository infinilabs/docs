---
title: "使用 INFINI Console 管理"
date: 0001-01-01
description: "通过 INFINI Console 统一管理一个或多个 Easysearch 集群，提供监控、告警、数据探索等企业级功能。"
summary: "使用 INFINI Console 管理 #  如果你需要多集群统一管理、告警通知、数据探索等企业级功能，推荐使用 INFINI Console。它是 INFINI Labs 提供的独立可视化管理平台，支持同时管理 Easysearch 和 Elasticsearch 集群。
 💡 如果你只需要管理单个集群并进行日常开发调试，Easysearch 还提供了 内置 Web UI，零部署，开箱即用。
 安装 Console #  方式一：一键启动（推荐） #  同时启动 Console + Easysearch，最快体验完整管理功能：
curl -fsSL http://get.infini.cloud/start-local | sh -s 支持更多参数：
# 启动 3 个 Easysearch 节点，自定义密码，开启 Agent 指标采集 curl -fsSL http://get.infini.cloud/start-local | sh -s -- up \  --nodes 3 --password &#34;MyDevPass123.&#34; --metrics-agent 方式二：单独安装 #  # Linux 一键安装 curl -sSL http://get."
---


# 使用 INFINI Console 管理

如果你需要**多集群统一管理**、**告警通知**、**数据探索**等企业级功能，推荐使用 **INFINI Console**。它是 INFINI Labs 提供的独立可视化管理平台，支持同时管理 Easysearch 和 Elasticsearch 集群。

> 💡 如果你只需要管理单个集群并进行日常开发调试，Easysearch 还提供了[内置 Web UI]({{< relref "./easysearch-ui.md" >}})，零部署，开箱即用。

## 安装 Console

### 方式一：一键启动（推荐）

同时启动 Console + Easysearch，最快体验完整管理功能：

```bash
curl -fsSL http://get.infini.cloud/start-local | sh -s
```

支持更多参数：

```bash
# 启动 3 个 Easysearch 节点，自定义密码，开启 Agent 指标采集
curl -fsSL http://get.infini.cloud/start-local | sh -s -- up \
  --nodes 3 --password "MyDevPass123." --metrics-agent
```

### 方式二：单独安装

```bash
# Linux 一键安装
curl -sSL http://get.infini.cloud | bash -s -- -p console
cd /opt/console && ./console
```

### 方式三：Docker

```bash
docker run -d --name console \
  -p 9000:9000 \
  infinilabs/console:latest
```

启动后，打开浏览器访问 `http://localhost:9000`。

## 连接 Easysearch 集群

1. 登录 Console 后，进入 **系统管理** → **集群管理**
2. 点击 **新建集群**
3. 填写信息：
   - **集群名称**：自定义，如 `my-easysearch`
   - **地址**：`https://localhost:9200`
   - **认证**：输入用户名和密码
4. 点击 **测试连接**，确认连通后保存

Console 支持跨引擎、跨版本、跨集群统一管理，你可以将多个 Easysearch 和 Elasticsearch 集群注册到同一个 Console 中集中管理。

## 仪表盘

登录后首页展示全局仪表盘：

- **告警/通知/待办**：未处理的告警事件一目了然
- **集群/节点/主机概览**：所有已注册集群的整体状态
- **存储概览**：各集群的磁盘和 JVM 使用情况
- **快捷入口**：集群注册、数据探索、告警管理、安全管理

## 集群监控

在集群管理页面可以查看：

- **集群列表**：健康状态、节点数、索引数、文档数、分片数
- **性能指标**：索引吞吐、搜索吞吐、索引延迟、查询延迟
- **筛选分组**：按健康状态、引擎版本、区域、标签筛选

点击具体集群可查看详细监控：

- **节点级别**：每个节点的 CPU / 内存 / JVM / 磁盘指标
- **索引级别**：每个索引的读写速率、文档增长趋势
- **集群动态**：实时事件流，如分片迁移、节点加入/离开

## 数据探索

Console 提供了类似 Kibana Discover 的数据探索功能：

1. 进入 **数据探索** 页面
2. 选择目标集群和索引
3. 浏览文档列表，查看字段详情
4. 支持基于字段值的快速过滤和搜索
5. 支持时序数据的时间范围选择

## 开发工具（DevTools）

Console 的开发工具在内置 UI 基础上进一步增强：

- **多集群 Tab**：同时打开多个集群的查询窗口，快速切换
- **常用命令收藏**：保存常用的查询语句，团队共享
- **SQL 查询**：直接编写 SQL，自动转换为 DSL 执行
- **语法高亮 + 自动补全**：API 路径和 JSON 字段名提示

```
GET /_cluster/health
```

```
GET /megacorp/_search
{
  "query": {
    "match": {
      "last_name": "Smith"
    }
  }
}
```

点击 ▶ 按钮或按 `Ctrl+Enter` 执行。

## 告警中心

Console 提供完善的告警能力：

- **预设规则**：集群健康状态变化、磁盘使用率、JVM 内存等常用告警规则开箱即用
- **多级别告警**：Info / Warning / Critical 分级，避免告警疲劳
- **多平台通知**：支持飞书、钉钉、企业微信、邮件、Slack、Discord、Webhook 等

## 安全管理

- **连接凭证管理**：集中管理各集群的访问凭证
- **用户授权**：Console 后台用户权限控制
- **审计日志**：记录所有操作行为，满足合规要求

## 平台监控

在平台概览页面可以查看所有已注册集群的汇总信息：

- 引擎分布（Easysearch / Elasticsearch 占比）
- JDK 版本分布
- 磁盘使用 Top 10 / JVM 使用 Top 10

## 功能概览

| 功能模块 | 说明 |
|----------|------|
| **仪表盘** | 告警通知、集群/节点/主机概览、快捷入口 |
| **集群管理** | 多集群列表、健康状态、索引/搜索速率、筛选分组 |
| **节点监控** | CPU/内存/JVM/磁盘指标、节点级别性能监控 |
| **索引监控** | 索引吞吐、查询延迟、索引级别指标 |
| **数据探索** | 类似 Kibana Discover，浏览文档、字段过滤、时序数据 |
| **告警中心** | 预设规则、多级别告警、多平台通知 |
| **开发工具** | 多集群 Tab、常用命令收藏、SQL 查询 |
| **安全管理** | 凭证管理、用户授权、审计日志 |

## 内置 UI vs Console 如何选择

| 场景 | 推荐方案 |
|------|----------|
| 单集群、个人开发/学习 | [内置 Easysearch UI]({{< relref "./easysearch-ui.md" >}}) |
| 多集群统一管理 | INFINI Console |
| 需要告警通知 | INFINI Console |
| 资源受限的边缘节点 | [内置 Easysearch UI]({{< relref "./easysearch-ui.md" >}}) |
| 企业级运维、审计合规 | INFINI Console |

## 下一步

- [使用 Easysearch 内置 UI]({{< relref "./easysearch-ui.md" >}})：零部署的轻量管理界面
- [使用 Curl 访问 Easysearch]({{< relref "./curl.md" >}})：命令行方式
- [Java 客户端]({{< relref "./java-client.md" >}})
- [Python 客户端]({{< relref "./python-client.md" >}})


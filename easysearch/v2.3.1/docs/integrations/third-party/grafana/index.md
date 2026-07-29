---
title: "Grafana 集成"
date: 0001-01-01
description: "将 Easysearch 作为数据源接入 Grafana，构建监控看板与数据可视化。"
summary: "Grafana 集成 #  Grafana 是主流的开源可观测平台，通过内置的 Elasticsearch 数据源插件可以直接连接 Easysearch，构建实时监控看板和数据分析面板。
相关指南 #    Superset 集成  监控告警  前置条件 #     条件 说明     Grafana 版本 8.0+ 推荐（内置 Elasticsearch 数据源）   网络可达 Grafana 服务器能够访问 Easysearch 的 9200 端口   认证信息 Easysearch 用户名和密码    配置步骤 #  1. 添加数据源 #   进入 Grafana → Configuration → Data Sources → Add data source 搜索并选择 Elasticsearch 填写连接信息：     配置项 值     URL https://easysearch-host:9200   Access Server（推荐）   Basic Auth 开启   User / Password Easysearch 用户名 / 密码   Skip TLS Verify 开发环境可开启；生产环境配置 CA 证书   Version 选择 7."
---


# Grafana 集成

Grafana 是主流的开源可观测平台，通过内置的 Elasticsearch 数据源插件可以直接连接 Easysearch，构建实时监控看板和数据分析面板。

## 相关指南

- [Superset 集成]({{< relref "../clients/superset.md" >}})
- [监控告警]({{< relref "../../operations/monitoring.md" >}})

## 前置条件

| 条件           | 说明                                             |
| -------------- | ------------------------------------------------ |
| Grafana 版本   | 8.0+ 推荐（内置 Elasticsearch 数据源）           |
| 网络可达       | Grafana 服务器能够访问 Easysearch 的 9200 端口    |
| 认证信息       | Easysearch 用户名和密码                          |

## 配置步骤

### 1. 添加数据源

1. 进入 Grafana → **Configuration → Data Sources → Add data source**
2. 搜索并选择 **Elasticsearch**
3. 填写连接信息：

| 配置项            | 值                                  |
| ----------------- | ----------------------------------- |
| URL               | `https://easysearch-host:9200`      |
| Access            | Server（推荐）                      |
| Basic Auth        | 开启                                |
| User / Password   | Easysearch 用户名 / 密码            |
| Skip TLS Verify   | 开发环境可开启；生产环境配置 CA 证书 |
| Version           | 选择 `7.10+`                        |
| Index name        | 要查询的索引模式，如 `logs-*`       |
| Time field name   | 时间字段，如 `@timestamp`           |
| Max concurrent Shard Requests | 建议 5                  |

### 2. 测试连接

点击 **Save & Test**，看到绿色提示 "Index OK. Time field name OK." 即表示连接成功。

## 常用面板类型

| 面板类型       | 适用场景                     | 对应查询能力                  |
| -------------- | ---------------------------- | ----------------------------- |
| Time Series    | 日志趋势、请求量、延迟监控   | `date_histogram` 聚合         |
| Bar Chart      | 分类统计、Top N 分析         | `terms` 聚合                  |
| Stat / Gauge   | KPI 指标、集群健康度         | `sum` / `avg` / `count`       |
| Table          | 明细数据查看、日志检索       | 原始文档查询                  |
| Logs           | 日志浏览与上下文查看         | 日志查询 + 高亮               |
| Heatmap        | 时间×维度交叉分析            | 多级聚合                      |

## 告警配置

Grafana 8+ 支持统一告警（Unified Alerting），可以基于 Easysearch 数据源的查询结果配置告警规则：

1. 在面板编辑页点击 **Alert** 标签
2. 设置告警条件（如：5 分钟内错误日志数 > 100）
3. 配置通知渠道（邮件、钉钉、Slack、Webhook 等）

## 性能建议

- **时间范围**：避免查询过大的时间跨度，建议默认看板时间范围不超过 24 小时
- **最小间隔**：`date_histogram` 的 `min_interval` 根据时间跨度合理设置（如 `1m`、`5m`）
- **专用账户**：为 Grafana 创建只读的专用账户，限制可访问的索引范围
- **查询缓存**：对于不频繁变化的数据，适当增大面板的刷新间隔


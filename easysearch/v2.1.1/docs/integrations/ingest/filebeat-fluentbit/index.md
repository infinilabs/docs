---
title: "轻量 Agent 接入：Filebeat / Fluent Bit"
date: 0001-01-01
description: "使用轻量日志 Agent 将日志采集到 Easysearch 或其前置缓冲层的推荐实践。"
summary: "轻量 Agent 接入：Filebeat / Fluent Bit #  Filebeat 和 Fluent Bit 是两款常用的轻量级日志采集 Agent，均可将日志数据直接发送到 Easysearch。
相关指南（先读这些） #    Logstash 接入  摄取管道  Agent 对比 #     特性 Filebeat Fluent Bit     语言 Go C   内存占用 ~30-50 MB ~5-10 MB   配置方式 YAML INI / YAML   输出到 ES ✅ 原生支持 ✅ 原生支持（es output）   Ingest Pipeline ✅ 支持指定 ✅ 支持指定   多行日志 ✅ multiline 模块 ✅ multiline parser   生态模块 丰富（modules for nginx等） 较少，但灵活的 parser 体系   适用场景 通用日志采集 边缘设备、资源受限环境    Filebeat 配置示例 #  基础配置 #  # filebeat."
---


# 轻量 Agent 接入：Filebeat / Fluent Bit

Filebeat 和 Fluent Bit 是两款常用的轻量级日志采集 Agent，均可将日志数据直接发送到 Easysearch。

## 相关指南（先读这些）

- [Logstash 接入]({{< relref "./logstash.md" >}})
- [摄取管道]({{< relref "/docs/features/ingest-pipelines/_index.md" >}})

## Agent 对比

| 特性             | Filebeat                    | Fluent Bit                    |
| ---------------- | --------------------------- | ----------------------------- |
| 语言             | Go                          | C                             |
| 内存占用         | ~30-50 MB                   | ~5-10 MB                      |
| 配置方式         | YAML                        | INI / YAML                    |
| 输出到 ES        | ✅ 原生支持                  | ✅ 原生支持（es output）       |
| Ingest Pipeline  | ✅ 支持指定                  | ✅ 支持指定                    |
| 多行日志         | ✅ multiline 模块            | ✅ multiline parser            |
| 生态模块         | 丰富（modules for nginx等）  | 较少，但灵活的 parser 体系     |
| 适用场景         | 通用日志采集                 | 边缘设备、资源受限环境         |

## Filebeat 配置示例

### 基础配置

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/app/*.log
    # 多行日志合并（如 Java 异常堆栈）
    multiline.pattern: '^\d{4}-\d{2}-\d{2}'
    multiline.negate: true
    multiline.match: after

output.elasticsearch:
  hosts: ["https://easysearch-node1:9200"]
  username: "admin"
  password: "your_password"
  ssl.verification_mode: "none"    # 生产环境应配置 CA 证书
  index: "app-logs-%{+yyyy.MM.dd}"
  pipeline: "app-log-pipeline"     # 可选：指定 Ingest Pipeline

# 索引模板设置
setup.template.name: "app-logs"
setup.template.pattern: "app-logs-*"
setup.ilm.enabled: false           # 按需开启 ILM
```

### 使用 Filebeat 模块

Filebeat 提供开箱即用的模块（如 nginx、system、mysql 等），可以自动解析常见日志格式：

```yaml
filebeat.modules:
  - module: nginx
    access:
      enabled: true
      var.paths: ["/var/log/nginx/access.log"]
    error:
      enabled: true

output.elasticsearch:
  hosts: ["https://easysearch-node1:9200"]
  username: "admin"
  password: "your_password"
```

## Fluent Bit 配置示例

```ini
# fluent-bit.conf
[SERVICE]
    Flush        5
    Log_Level    info

[INPUT]
    Name         tail
    Path         /var/log/app/*.log
    Tag          app.*
    Multiline    On
    Parser_Firstline  multiline_java

[FILTER]
    Name         record_modifier
    Match        app.*
    Record       hostname ${HOSTNAME}

[OUTPUT]
    Name         es
    Match        app.*
    Host         easysearch-node1
    Port         9200
    HTTP_User    admin
    HTTP_Passwd  your_password
    tls          On
    tls.verify   Off
    Index        app-logs
    Type         _doc
    Pipeline     app-log-pipeline
    Suppress_Type_Name  On
```

## 架构选型

根据数据量和可靠性要求，可以选择不同的接入架构：

```
方案 A：直写（简单，适合中小规模）
  Agent → Easysearch

方案 B：加缓冲（推荐，适合大规模）
  Agent → Kafka/Redis → Logstash → Easysearch

方案 C：网关模式
  Agent → INFINI Gateway → Easysearch
```

| 方案   | 优势                               | 劣势                          |
| ------ | ---------------------------------- | ----------------------------- |
| 直写   | 架构简单，延迟低                   | 流量突增可能压垮 Easysearch    |
| 加缓冲 | 流量削峰、数据不丢失               | 架构复杂，延迟增加             |
| 网关   | 限流、路由、格式转换               | 需要部署 Gateway 组件          |

## 生产建议

- 始终配置 **死信队列**（DLQ），避免解析失败的日志丢失
- 设置合理的 **batch size** 和 **flush interval**，平衡延迟与吞吐
- 使用 **Ingest Pipeline** 在 Easysearch 端做字段提取、时间解析等预处理
- 为日志索引配置 **索引生命周期管理（ILM）** 或定期清理策略
- 监控 Agent 的内存和 CPU 使用，避免采集端成为瓶颈



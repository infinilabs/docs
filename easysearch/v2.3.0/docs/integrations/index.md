---
title: "生态集成"
date: 0001-01-01
description: "官方客户端与第三方系统集成说明。"
summary: "生态集成 #  Easysearch 与上下游系统的集成指南，涵盖客户端接入、数据管道、AI 工作流、安全认证、可观测性与第三方框架。
数据接入 #  将各类数据源稳定地接入 Easysearch：
  Logstash 日志接入：Pipeline 配置与插件使用  Filebeat / Fluent Bit：轻量级日志采集 Agent 对比与配置  数据库同步（JDBC / ETL）：增量同步、CDC 与 ETL 方案  客户端与 SDK #  面向不同语言和场景的连接方式：
  Java 客户端：REST High Level Client 使用与生产配置  SQL / JDBC 查询：用 SQL 语法查询 Easysearch  Superset 集成：BI 看板与数据可视化  AI 与向量检索 #  让 Easysearch 成为 AI 工作流中的检索中枢：
  Embedding 服务接入：对接文本向量化服务  向量工作流与 Hybrid 检索：向量写入与 BM25 + kNN 组合  RAG 与 LLM 集成：构建检索增强生成应用  安全与权限 #  将 Easysearch 纳入企业安全体系："
---


# 生态集成

Easysearch 与上下游系统的集成指南，涵盖客户端接入、数据管道、AI 工作流、安全认证、可观测性与第三方框架。

## 数据接入

将各类数据源稳定地接入 Easysearch：

- [Logstash 日志接入]({{< relref "./ingest/logstash.md" >}})：Pipeline 配置与插件使用
- [Filebeat / Fluent Bit]({{< relref "./ingest/filebeat-fluentbit.md" >}})：轻量级日志采集 Agent 对比与配置
- [数据库同步（JDBC / ETL）]({{< relref "./ingest/jdbc-etl.md" >}})：增量同步、CDC 与 ETL 方案

## 客户端与 SDK

面向不同语言和场景的连接方式：

- [Java 客户端]({{< relref "./clients/java.md" >}})：REST High Level Client 使用与生产配置
- [SQL / JDBC 查询]({{< relref "./clients/sql.md" >}})：用 SQL 语法查询 Easysearch
- [Superset 集成]({{< relref "./clients/superset.md" >}})：BI 看板与数据可视化

## AI 与向量检索

让 Easysearch 成为 AI 工作流中的检索中枢：

- [Embedding 服务接入]({{< relref "./ai/embedding-service.md" >}})：对接文本向量化服务
- [向量工作流与 Hybrid 检索]({{< relref "./ai/vector-workflow.md" >}})：向量写入与 BM25 + kNN 组合
- [RAG 与 LLM 集成]({{< relref "./ai/rag-and-llm.md" >}})：构建检索增强生成应用

## 安全与权限

将 Easysearch 纳入企业安全体系：

- [接入企业认证体系]({{< relref "./security/auth-integration.md" >}})：LDAP / OIDC / SSO 集成
- [多租户与权限模型]({{< relref "./security/multi-tenant-access-control.md" >}})：索引级、文档级、字段级安全

## 可观测性

APM、监控与 Trace 数据存储：

- [OpenTelemetry 集成]({{< relref "./observability/opentelemetry.md" >}})：标准化可观测性数据接入
- [SkyWalking 存储后端]({{< relref "./observability/skywalking.md" >}})：APM 数据存储配置

## 插件扩展

- [插件开发总览]({{< relref "./plugins/_index.md" >}})：从入门到发布的完整插件开发文档
- [插件开发入门]({{< relref "./plugins/getting-started.md" >}})：从模板开始快速完成第一个插件

## 第三方框架

- [Easy-ES 查询框架]({{< relref "./third-party/easy-es.md" >}})
- [ORM 框架集成]({{< relref "./third-party/orm.md" >}})
- [LangChain 集成]({{< relref "./third-party/langchain.md" >}})
- [SeaTunnel 集成]({{< relref "./third-party/seatunnel.md" >}})
- [Grafana 集成]({{< relref "./third-party/grafana.md" >}})

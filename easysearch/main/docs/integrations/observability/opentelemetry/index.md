---
title: "OpenTelemetry 集成"
date: 0001-01-01
description: "将 OTel 指标、日志与 Trace 落地到 Easysearch，构建统一可观测性存储。"
summary: "OpenTelemetry 集成 #  本页从架构和建模角度，讲清楚一件事：如果你已经用了 OpenTelemetry，怎么把指标/日志/Trace 稳定地落到 Easysearch 上，并跑起来统一分析？
 数据流：应用 → OTel SDK → OTel Collector → Easysearch 索引与 mapping：metrics / logs / traces 三类数据怎么按索引和字段建模 与现有监控/告警体系的分工：谁做实时告警，谁做长期存储与分析  不涉及具体 Collector 配置语法的全集，只给出 Easysearch 侧的“接入契约”与推荐套路。
1. OTel → Easysearch 的典型数据流 #  一个常见部署大致长这样：
 应用侧：接入 OTel SDK（或通过语言/框架集成），上报 metrics / logs / traces OTel Collector：集中收集、处理、转发 OTel 数据 输出到 Easysearch：  通过支持 Easysearch/ES 协议的 exporter 或通过自定义 exporter / gateway，将数据转换为 Easysearch 可接受的 HTTP 请求（通常是 _bulk）    在 Easysearch 看来，本质上就是有一条“写入大量结构化/半结构化数据”的管道，区别只在于字段和索引设计。"
---


# OpenTelemetry 集成

本页从架构和建模角度，讲清楚一件事：**如果你已经用了 OpenTelemetry，怎么把指标/日志/Trace 稳定地落到 Easysearch 上，并跑起来统一分析？**

- 数据流：应用 → OTel SDK → OTel Collector → Easysearch
- 索引与 mapping：metrics / logs / traces 三类数据怎么按索引和字段建模
- 与现有监控/告警体系的分工：谁做实时告警，谁做长期存储与分析

不涉及具体 Collector 配置语法的全集，只给出 Easysearch 侧的“接入契约”与推荐套路。

## 1. OTel → Easysearch 的典型数据流

一个常见部署大致长这样：

1. **应用侧**：接入 OTel SDK（或通过语言/框架集成），上报 metrics / logs / traces
2. **OTel Collector**：集中收集、处理、转发 OTel 数据
3. **输出到 Easysearch**：
   - 通过支持 Easysearch/ES 协议的 exporter
   - 或通过自定义 exporter / gateway，将数据转换为 Easysearch 可接受的 HTTP 请求（通常是 `_bulk`）

在 Easysearch 看来，本质上就是有一条“写入大量结构化/半结构化数据”的管道，区别只在于字段和索引设计。

## 2. 索引设计：按“信号类型 + 时间”切分

建议把 OTel 的三类信号拆成至少三块索引前缀：

- 指标（metrics）：如 `otel-metrics-YYYY.MM.DD`
- 日志（logs）：如 `otel-logs-YYYY.MM.DD`
- Trace / Span：如 `otel-traces-YYYY.MM.DD`

理由：

- 不同信号有不同的查询模式和字段布局，混在一个索引里会让 mapping 很难控制
- 按时间切分索引，方便做数据保留策略与冷热分层
- 后续你也可以根据业务再细分（例如按环境/业务域再拆前缀）

可以结合索引模板，让新创建的 `otel-*` 索引自动带上合适的 settings / mappings / aliases。

## 3. 字段与 mapping 设计要点

OpenTelemetry 的语义里已经定义了很多标准字段，建议在 Easysearch 侧尽量复用这些名字，不要“自创一套”：

- **共通字段：**
  - `service.name`、`service.namespace`、`service.instance.id`
  - `resource.*`：环境、集群、节点、region 等元信息
- **指标（metrics）：**
  - `metric.name`、`metric.type`（counter/gauge/histogram 等）
  - `metric.value` 或直方图相关字段
  - 标签（dimension）统一放在 `attributes.*` 下（例如 `attributes.http.method`）
- **日志（logs）：**
  - `body`：原始日志内容（字符串或结构化）
  - `severity` 或 `log.level`
  - 关联的 `trace_id` / `span_id`（方便从日志跳转到 Trace）
- **Trace / Span：**
  - `trace_id`、`span_id`、`parent_span_id`
  - `span.name`、`span.kind`、`status.code`
  - `duration_ms`（自算一个便于聚合的数值字段）
  - 关键 attributes（如 http.url、db.statement 等）仍放在 `attributes.*`

结合 Easysearch 的 mapping 能力：

- 对于高基数的标识类字段（trace_id / span_id / service.instance.id 等），使用 `keyword` 类型
- 对时间戳统一用 `@timestamp`（`date` 类型），方便跨类型联合分析
- 对一部分常用标签，可以单独“抄一份”到顶层字段（如 `env`、`region`），减少查询时的嵌套路径开销

## 4. 典型查询与聚合场景

把数据落在 Easysearch 后，常见会有几类查询/分析需求：

- **指标看板：**
  - 对 `metric.value` 按 `date_histogram` 聚合，按 `service.name` 等分组
  - 和一般的时间序列指标几乎一致，只是字段命名更贴近 OTel 语义
- **日志+Trace 联动：**
  - 在日志索引里根据 `trace_id` / `span_id` 找到相关日志
  - 在 Trace 索引里根据 `trace_id` 做整条调用链的重建与聚合分析
- **多维 Drill-down：**
  - 利用 `attributes.*` 里的标签，按照 http.method、status_code、region 等做分组统计
  - 在高维标签场景下，需要关注字段数量和高基数字段对聚合性能的影响

这些查询模式本身和“普通的日志/指标看板”非常接近，只是字段命名和建模遵循了 OTel 的规范。

## 5. 与现有监控/告警平台的分工

在很多团队里，告警往往由 Prometheus / 专用 APM / SaaS 平台承担，而 Easysearch 更适合作为：

- **统一检索层**：日志、Trace、部分指标统一进来做问题排查和历史分析
- **长周期存储层**：将 OTel 数据的长期保留交给 Easysearch，短周期/实时告警仍放在专门的时序系统

这种分工下：

- 实时告警：继续用现有告警系统（如 Prometheus rules / Alertmanager）
- 事后分析与追溯：通过 Easysearch + 可视化（如 Superset、自研控制台等）做多维查询和分析

Easysearch 侧需要重点做好：

- 数据保留策略（参考“数据生命周期与保留策略”一文）
- 索引与分片设计（特别是高写入量的 Trace 与日志索引）
- 安全与多租户（按环境/团队/服务做权限和视图隔离）

## 6. 相关文档

可以搭配阅读以下章节，形成一条完整的 OTel → Easysearch 可观测性链路：

- [索引模板]({{< relref "/docs/operations/data-management/index-templates" >}})：为 OTel 指标/日志/Trace 定义索引模板
- [时间序列建模]({{< relref "/docs/best-practices/data-modeling/time-series" >}})：时序数据的索引设计与查询优化
- [数据生命周期]({{< relref "/docs/best-practices/data-lifecycle" >}})：指标/日志/Trace 的保留与归档策略
- [索引设计]({{< relref "/docs/best-practices/index-design" >}})：高写入场景下的索引/分片规划


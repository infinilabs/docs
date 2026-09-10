---
title: "集群监控"
date: 0001-01-01
description: "关键指标、健康检查、监控工具（Easysearch-UI、INFINI Console）。"
summary: "监控 #  本文从&quot;如何判断集群是否健康&quot;出发，给出最小可行的监控指标与告警建议，并提供具体的 API 示例。
 监控三层模型 #  Easysearch 集群的监控可以分为三个层次：
┌─────────────────────────────────────────────────────────────┐ │ Layer 1: 服务可用性 │ │ • 集群是否整体可用？节点是否存活？ │ ├─────────────────────────────────────────────────────────────┤ │ Layer 2: 资源与性能 │ │ • CPU/内存/磁盘/网络是否有瓶颈？延迟是否异常？ │ ├─────────────────────────────────────────────────────────────┤ │ Layer 3: 业务行为 │ │ • 慢查询、错误率、索引状态是否正常？ │ └─────────────────────────────────────────────────────────────┘ 监控目标：
 出问题前能看到&quot;趋势变坏&quot;（而不是只在宕机后才收到告警） 告警有明确的处理指引，而不是一堆噪音   集群健康检查 #  快速健康检查 #  GET _cluster/health 响应示例：
{ &#34;cluster_name&#34;: &#34;production&#34;, &#34;status&#34;: &#34;green&#34;, &#34;timed_out&#34;: false, &#34;number_of_nodes&#34;: 5, &#34;number_of_data_nodes&#34;: 3, &#34;active_primary_shards&#34;: 50, &#34;active_shards&#34;: 100, &#34;relocating_shards&#34;: 0, &#34;initializing_shards&#34;: 0, &#34;unassigned_shards&#34;: 0, &#34;delayed_unassigned_shards&#34;: 0, &#34;number_of_pending_tasks&#34;: 0, &#34;number_of_in_flight_fetch&#34;: 0, &#34;task_max_waiting_in_queue_millis&#34;: 0, &#34;active_shards_percent_as_number&#34;: 100."
---


# 监控

本文从"如何判断集群是否健康"出发，给出最小可行的监控指标与告警建议，并提供具体的 API 示例。

---

## 监控三层模型

Easysearch 集群的监控可以分为三个层次：

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 1: 服务可用性                                         │
│  • 集群是否整体可用？节点是否存活？                            │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: 资源与性能                                         │
│  • CPU/内存/磁盘/网络是否有瓶颈？延迟是否异常？                 │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: 业务行为                                           │
│  • 慢查询、错误率、索引状态是否正常？                          │
└─────────────────────────────────────────────────────────────┘
```

**监控目标**：
- 出问题前能看到"趋势变坏"（而不是只在宕机后才收到告警）
- 告警有明确的处理指引，而不是一堆噪音

---

## 集群健康检查

### 快速健康检查

```json
GET _cluster/health
```

**响应示例**：
```json
{
  "cluster_name": "production",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 5,
  "number_of_data_nodes": 3,
  "active_primary_shards": 50,
  "active_shards": 100,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100.0
}
```

**状态含义**：

| 状态 | 含义 | 处理建议 |
|------|------|----------|
| **green** | 所有分片（主分片+副本）均已分配 | 正常状态 |
| **yellow** | 所有主分片已分配，但部分副本未分配 | 检查节点数量是否满足副本需求 |
| **red** | 部分主分片未分配，数据可能丢失 | **立即处理**，检查未分配原因 |

### 等待集群健康

在自动化脚本中，等待集群达到特定状态：

```json
GET _cluster/health?wait_for_status=green&timeout=60s
```

### 索引级健康检查

```json
GET _cluster/health/logs-*?level=indices
```

---

## 关键监控指标

### 集群级指标

```json
GET _cluster/stats
```

**核心关注点**：

| 指标 | 路径 | 告警阈值建议 |
|------|------|-------------|
| 节点数 | `nodes.count.total` | 低于预期节点数 |
| 分片总数 | `indices.shards.total` | 单节点 >1000 个分片 |
| 文档数 | `indices.docs.count` | 趋势监控 |
| 存储大小 | `indices.store.size_in_bytes` | 趋势监控 |

### 节点级指标

```json
GET _nodes/stats
```

**简化视图**（推荐日常使用）：

```bash
GET _cat/nodes?v&h=name,heap.percent,ram.percent,cpu,load_1m,disk.used_percent,node.role
```

**输出示例**：
```
name          heap.percent ram.percent cpu load_1m disk.used_percent node.role
node-1                  45          78  12    1.25              62.3 dim
node-2                  52          82  15    1.45              58.7 dim
node-master             23          45   5    0.32              25.1 m
```

**核心指标详解**：

| 指标 | 说明 | 告警阈值 |
|------|------|----------|
| `heap.percent` | JVM 堆使用率 | > 75% 警告，> 85% 严重 |
| `ram.percent` | 系统内存使用率 | > 90% 警告 |
| `cpu` | CPU 使用率 | > 80% 持续 5 分钟 |
| `load_1m` | 1 分钟负载 | > CPU 核心数 × 2 |
| `disk.used_percent` | 磁盘使用率 | > 75% 警告，> 85% 严重 |

### 磁盘水位监控

Easysearch 内置磁盘保护机制：

| 水位 | 默认值 | 行为 |
|------|--------|------|
| `low` | 85% | 不再向该节点分配新分片 |
| `high` | 90% | 尝试将分片迁移到其他节点 |
| `flood_stage` | 95% | 强制将受影响索引设为只读 |

**查看当前设置**：
```json
GET _cluster/settings?include_defaults=true&filter_path=*.cluster.routing.allocation.disk*
```

### 分片级指标

```bash
# 分片分布概览
GET _cat/shards?v&s=state,index

# 只看问题分片
GET _cat/shards?v&h=index,shard,prirep,state,unassigned.reason&s=state:desc
```

**未分配分片诊断**：

```json
GET _cluster/allocation/explain
```

**响应示例**（磁盘不足）：
```json
{
  "index": "logs-2026.02.11",
  "shard": 0,
  "primary": true,
  "current_state": "unassigned",
  "unassigned_info": {
    "reason": "INDEX_CREATED",
    "at": "2026-02-11T10:00:00.000Z"
  },
  "can_allocate": "no",
  "allocate_explanation": "cannot allocate because allocation is not permitted to any of the nodes",
  "node_allocation_decisions": [
    {
      "node_name": "node-1",
      "node_decision": "no",
      "deciders": [
        {
          "decider": "disk_threshold",
          "decision": "NO",
          "explanation": "the node is above the high watermark cluster setting"
        }
      ]
    }
  ]
}
```

---

## 性能指标监控

### 搜索性能

```json
GET _nodes/stats/indices/search
```

**关键指标**：

| 指标 | 路径 | 说明 |
|------|------|------|
| 搜索总数 | `indices.search.query_total` | QPS = 差值 / 时间间隔 |
| 搜索耗时 | `indices.search.query_time_in_millis` | 平均延迟 = 耗时 / 总数 |
| 当前搜索 | `indices.search.query_current` | 并发搜索数 |
| Fetch 耗时 | `indices.search.fetch_time_in_millis` | 结果获取耗时 |

### 写入性能

```json
GET _nodes/stats/indices/indexing
```

**关键指标**：

| 指标 | 路径 | 说明 |
|------|------|------|
| 写入总数 | `indices.indexing.index_total` | 写入 QPS |
| 写入耗时 | `indices.indexing.index_time_in_millis` | 写入延迟 |
| 写入失败 | `indices.indexing.index_failed` | 应为 0 |

### Bulk 拒绝监控

```json
GET _nodes/stats/thread_pool/write
```

**关键指标**：

| 指标 | 说明 | 告警条件 |
|------|------|----------|
| `rejected` | 被拒绝的请求数 | > 0 且持续增长 |
| `queue` | 当前队列大小 | 接近最大值 |

### 段合并监控

```json
GET _nodes/stats/indices/merges
```

**关注点**：
- `current`：当前合并数
- `total_time_in_millis`：合并总耗时
- 合并压力过大会影响写入和查询性能

---

## 慢查询日志

### 配置慢查询阈值

```json
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s",
  "index.search.slowlog.threshold.query.debug": "2s",
  "index.search.slowlog.threshold.query.trace": "500ms",
  "index.search.slowlog.threshold.fetch.warn": "1s",
  "index.search.slowlog.threshold.fetch.info": "800ms"
}
```

### 配置慢写入阈值

```json
PUT /my-index/_settings
{
  "index.indexing.slowlog.threshold.index.warn": "10s",
  "index.indexing.slowlog.threshold.index.info": "5s"
}
```

### 查看慢查询日志

日志位置：`logs/{cluster_name}_index_search_slowlog.log`

**日志格式示例**：
```
[2026-02-11T10:30:00,123][WARN][index.search.slowlog.query] [node-1] [logs-2026.02.11][0] took[12.3s], took_millis[12345], total_hits[10000], types[], stats[], search_type[QUERY_THEN_FETCH], total_shards[5], source[{"query":{"match_all":{}}}]
```

---

## 常见告警规则

### 可用性告警

| 告警名称 | 条件 | 级别 | 处理建议 |
|----------|------|------|----------|
| 集群状态异常 | `status != green` 持续 5 分钟 | P1 | 检查分片分配 |
| 节点离线 | 节点数 < 预期 | P1 | 检查节点进程 |
| 未分配分片 | `unassigned_shards > 0` 持续 10 分钟 | P2 | 诊断分配原因 |

### 资源告警

| 告警名称 | 条件 | 级别 | 处理建议 |
|----------|------|------|----------|
| 磁盘空间不足 | `disk.used_percent > 85%` | P2 | 清理数据或扩容 |
| 磁盘空间严重不足 | `disk.used_percent > 90%` | P1 | 立即释放空间 |
| JVM 堆压力 | `heap.percent > 85%` 持续 10 分钟 | P2 | 检查查询或扩容 |
| CPU 过载 | `cpu > 90%` 持续 5 分钟 | P2 | 检查热点查询 |

### 性能告警

| 告警名称 | 条件 | 级别 | 处理建议 |
|----------|------|------|----------|
| 搜索延迟升高 | P95 > 基线 × 2 | P3 | 检查慢查询 |
| Bulk 拒绝 | `rejected > 0` 且持续增长 | P2 | 降低写入并发 |
| 段合并积压 | `current_merges > 5` 持续 | P3 | 检查写入模式 |

---

## 监控系统集成

### Prometheus Exporter 方式

使用社区 Exporter 将指标暴露给 Prometheus：

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'easysearch'
    static_configs:
      - targets: ['es-exporter:9114']
```

### 定时拉取脚本方式

```bash
#!/bin/bash
# 每分钟采集关键指标

CLUSTER_HEALTH=$(curl -s "http://localhost:9200/_cluster/health")
NODE_STATS=$(curl -s "http://localhost:9200/_cat/nodes?format=json")

# 解析并发送到监控系统
STATUS=$(echo $CLUSTER_HEALTH | jq -r '.status')
UNASSIGNED=$(echo $CLUSTER_HEALTH | jq -r '.unassigned_shards')

# 示例：发送到 InfluxDB
curl -XPOST 'http://influxdb:8086/write?db=easysearch' \
  --data-binary "cluster_health,cluster=prod status=\"$STATUS\",unassigned=$UNASSIGNED"
```

### 日志采集

配置 Filebeat 采集 Easysearch 日志：

```yaml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/easysearch/*.log
    multiline.pattern: '^\['
    multiline.negate: true
    multiline.match: after
    
output.elasticsearch:
  hosts: ["monitoring-cluster:9200"]
  index: "easysearch-logs-%{+yyyy.MM.dd}"
```

---

## 监控仪表盘建议

### 概览面板

- 集群状态（green/yellow/red）
- 节点数量与角色分布
- 分片总数与未分配数
- 存储总量与可用空间

### 性能面板

- 搜索 QPS 与延迟（P50/P95/P99）
- 写入 QPS 与延迟
- Bulk 队列深度与拒绝数
- 段合并数与耗时

### 资源面板

- 各节点 CPU 使用率
- 各节点 JVM 堆使用率
- 各节点磁盘使用率
- 网络流量（入/出）

### 热点面板

- Top 10 慢查询索引
- Top 10 大索引（按存储）
- Top 10 热点分片

---

## 使用 INFINI Console 进行可视化监控

除了手动通过 API 查询监控数据，推荐使用 **INFINI Console** 来实现集群的可视化管理和监控。INFINI Console 是一款轻量级的多集群、跨版本搜索基础设施统一纳管平台。

### INFINI Console 简介

INFINI Console 提供了以下核心功能：

- **多集群管理**：统一管理多个 Easysearch/Elasticsearch 集群（支持跨版本）
- **可视化监控**：实时监控集群、节点、索引的关键指标
- **性能分析**：TopN 功能快速识别排名前 N 的关键指标数据点
- **数据管理**：索引管理、快照备份、数据迁移等操作
- **告警管理**：配置告警规则并接收通知

### 快速开始

**在线体验**：

访问 [INFINI Console Demo](https://demo.infini.cloud)（用户名/密码：`readonly/readonly`）

**本地部署**：

INFINI Console 支持 Docker、Kubernetes 等多种部署方式。具体部署指南请参考 [INFINI Console 官方文档](https://docs.infinilabs.com/console/)。

### 推荐的监控仪表盘

在 INFINI Console 中建议配置以下仪表盘：

1. **集群概览面板**：集群状态、节点数、分片分布、存储情况
2. **性能监控面板**：搜索/写入 QPS、延迟、Bulk 拒绝情况
3. **资源监控面板**：CPU、内存、JVM 堆、磁盘使用率
4. **告警面板**：实时告警、告警历史、处理记录
5. **慢查询分析面板**：Top 慢查询、错误率分析

---

## 使用 Easysearch-UI 进行集群管理

**Easysearch-UI** 是 Easysearch **v1.15.0+** 内置的官方轻量级管理界面，无需额外安装，开箱即用。

### 快速开始

启动 Easysearch 集群后，访问以下地址即可使用：

```
https://localhost:9200/_ui/
```

### 核心功能

Easysearch-UI 提供了以下功能：

#### 1. **节点管理**
- 查看集群中所有节点的信息
- 监控节点的状态和健康度
- 节点角色配置查看

#### 2. **索引管理**（功能丰富）
- 查看所有索引的详细信息
- 创建、删除、编辑索引
- 查看索引配置和映射
- **索引限流设置**：灵活配置索引资源配额，让硬件资源向高价值索引倾斜

#### 3. **分片管理**
- 查看分片分布情况
- **分片移动**：轻松实现分片重新分配和负载均衡
- 支持手动迁移分片到指定节点

#### 4. **模板管理**
- UI 界面引导创建和编辑索引模板
- 管理分量级别的模板配置

#### 5. **热点线程分析**
- 实时捕捉热点线程
- 可视化展示 CPU 消耗最高的线程
- 快速定位性能瓶颈

#### 6. **生命周期管理**
- 可视化配置 ILM（索引生命周期管理）策略
- 轻松管理索引的热温冷归档

#### 7. **备份管理**
- 配置和管理备份仓库
- 创建、查看、恢复快照
- 一键快照操作

#### 8. **Dev Tools**
- 类似 Kibana 的 Console，支持 REST API 调用
- 支持 DSL 查询编写和测试
- 实时查询结果展示

### 使用示例

#### 查看集群概览

登录后首页展示：
- 集群整体健康状态
- 节点数量和分布
- 主分片和副本分片统计
- 索引总数和文档总数

#### 快速索引操作

1. 进入"索引管理"
2. 查看索引统计：文档数、存储大小、分片配置
3. 支持批量操作：刷新、关闭、打开索引

#### 性能诊断

1. 进入"热点线程"
2. 选择时间窗口（如 30 秒）
3. 快速发现 CPU 消耗最高的操作

#### 容灾配置

1. 进入"备份管理" → "仓库配置"
2. 配置备份仓库（支持 HDFS、NFS、S3 等）
3. 创建快照实现数据备份和灾难恢复

---

## INFINI Console vs Easysearch-UI 对比

| 功能 | INFINI Console | Easysearch-UI |
|------|-----------------|---------------|
| **访问方式** | 独立应用，多集群 | 内置在 Easysearch，无需额外部署 |
| 多集群管理 | ✅ 支持多集群中央管理 | ❌ 单集群 |
| 仪表盘 | ✅ 丰富的自定义仪表盘 | ✅ 内置集群概览 |
| 告警与通知 | ✅ 完整的告警系统 | ❌ 不支持 |
| 性能分析 | ✅ TopN 性能分析、实时监控 | ✅ 热点线程捕捉 |
| 索引管理 | ✅ 支持 | ✅ 完整的索引管理（含限流、分片移动） |
| 生命周期管理 | ✅ ILM 管理 | ✅ 可视化 ILM 配置 |
| 备份恢复 | ✅ 支持 | ✅ 快照管理 |
| 数据迁移 | ✅ 高级数据迁移工具 | ❌ 不支持 |
| Dev Tools | ✅ 支持 | ✅ 支持（REST API + DSL） |
| 部署成本 | 需要额外部署（Docker/K8s/二进制） | 零部署成本（内置） |

**选择建议**：
- **快速开始、集群基础管理**：使用 Easysearch-UI（v1.15.0+）
- **专业监控、告警、多集群管理**：使用 INFINI Console
- **最佳实践**：Easysearch-UI 作为集群内置管理工具，INFINI Console 作为企业级监控平台，两者结合使用可实现完整的可观测性

---

## 小结

| 层次 | 关键指标 | 主要 API |
|------|----------|----------|
| 可用性 | 集群状态、节点数、未分配分片 | `_cluster/health` |
| 资源 | CPU、内存、磁盘、JVM 堆 | `_nodes/stats`、`_cat/nodes` |
| 性能 | 搜索/写入延迟、Bulk 拒绝 | `_nodes/stats/indices` |
| 行为 | 慢查询、错误率 | 慢查询日志 |

**下一步阅读**：
- [故障排查]({{< relref "./troubleshooting" >}})：面对告警时的诊断路径
- [容量规划]({{< relref "/docs/best-practices/index-design.md" >}})：在问题发生前预留足够余量
- [备份还原]({{< relref "/docs/features/data-retention/backup-restore" >}})：数据安全的最后一道防线


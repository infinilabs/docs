---
title: "SkyWalking 存储后端集成"
date: 0001-01-01
description: "将 SkyWalking 的指标与 Trace 存储后端指向 Easysearch 的配置与索引规划。"
summary: "SkyWalking 存储后端集成 #  Apache SkyWalking 是一款广泛使用的分布式应用性能监控（APM）系统。Easysearch 可以作为其存储后端，存储链路追踪和指标数据。
相关指南（先读这些） #    OpenTelemetry 集成  集群监控  架构说明 #  应用服务（Agent） ↓ gRPC / HTTP SkyWalking OAP Server ↓ ES 协议 Easysearch（存储后端） ↓ SkyWalking UI / Grafana（可视化） SkyWalking OAP Server 使用 Elasticsearch 协议写入和查询数据。由于 Easysearch 兼容 ES 7.x API，可以直接配置为存储后端。
配置步骤 #  1. 修改 OAP Server 配置 #  编辑 application.yml，将存储类型设为 elasticsearch：
storage: selector: elasticsearch elasticsearch: namespace: sw clusterNodes: https://easysearch-node1:9200 protocol: https user: admin password: your_password trustStorePath: /path/to/truststore."
---


# SkyWalking 存储后端集成

Apache SkyWalking 是一款广泛使用的分布式应用性能监控（APM）系统。Easysearch 可以作为其存储后端，存储链路追踪和指标数据。

## 相关指南（先读这些）

- [OpenTelemetry 集成]({{< relref "./opentelemetry.md" >}})
- [集群监控]({{< relref "../../operations/monitoring.md" >}})

## 架构说明

```
应用服务（Agent）
      ↓ gRPC / HTTP
SkyWalking OAP Server
      ↓ ES 协议
Easysearch（存储后端）
      ↓
SkyWalking UI / Grafana（可视化）
```

SkyWalking OAP Server 使用 Elasticsearch 协议写入和查询数据。由于 Easysearch 兼容 ES 7.x API，可以直接配置为存储后端。

## 配置步骤

### 1. 修改 OAP Server 配置

编辑 `application.yml`，将存储类型设为 `elasticsearch`：

```yaml
storage:
  selector: elasticsearch
  elasticsearch:
    namespace: sw
    clusterNodes: https://easysearch-node1:9200
    protocol: https
    user: admin
    password: your_password
    trustStorePath: /path/to/truststore.jks
    trustStorePass: changeit
    # 索引分片配置
    indexShardsNumber: 2
    indexReplicasNumber: 1
    # 数据保留天数
    recordDataTTL: 7
    metricsDataTTL: 30
    # 批量写入设置
    bulkActions: 5000
    flushInterval: 15
    concurrentRequests: 2
```

### 2. 关键参数说明

| 参数                  | 说明                                               | 推荐值       |
| --------------------- | -------------------------------------------------- | ------------ |
| `clusterNodes`        | Easysearch 节点地址                                 | 所有节点地址 |
| `indexShardsNumber`   | 每个索引的分片数                                    | 2~3          |
| `indexReplicasNumber` | 副本数                                              | 1            |
| `recordDataTTL`       | Trace 记录保留天数                                  | 3~7          |
| `metricsDataTTL`      | 指标数据保留天数                                    | 7~30         |
| `bulkActions`         | 批量写入操作数                                      | 5000         |
| `flushInterval`       | 刷新间隔（秒）                                      | 15           |

### 3. 验证连接

启动 OAP Server 后，检查日志确认连接成功：

```
grep "Elasticsearch storage" oap.log
```

在 Easysearch 中可以看到 SkyWalking 创建的索引：

```
GET _cat/indices/sw_*?v&s=index
```

## 容量规划

| 应用规模          | 日均 Trace 量  | 建议节点数 | 建议磁盘空间/节点 |
| ----------------- | -------------- | ---------- | ------------------ |
| 小型（<10 服务）  | <100 万        | 3 节点     | 100 GB             |
| 中型（10-50 服务）| 100-1000 万    | 3~5 节点   | 500 GB             |
| 大型（>50 服务）  | >1000 万       | 5+ 节点    | 1 TB+              |

## 优化建议

- **采样率**：在高流量场景下配置 SkyWalking Agent 的采样率（如 10%），减少数据量
- **索引生命周期**：配合 Easysearch 的 ILM 策略自动清理过期索引
- **Cold 存储**：历史数据可以迁移到低成本存储节点
- **独立集群**：生产环境建议将 APM 数据存储在独立的 Easysearch 集群中，避免影响业务搜索



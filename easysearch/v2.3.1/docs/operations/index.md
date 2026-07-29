---
title: "运维手册"
date: 0001-01-01
description: "容量规划、安全配置、监控告警、备份还原、集群扩容、故障排查——生产环境完整运维指南。"
summary: "运维手册 #  面向生产环境的完整运维指南，帮助您管理和维护 Easysearch 集群的稳定性、安全性和性能。
 运维全景图 #  ┌─────────────────────────────────────────────────────────────────────────┐ │ Easysearch 运维体系 │ ├─────────────────────────────────────────────────────────────────────────┤ │ │ │ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │ │ │ 规划部署 │───▶│ 日常运维 │───▶│ 故障处理 │ │ │ └─────────────┘ └─────────────┘ └─────────────┘ │ │ │ │ │ │ │ ▼ ▼ ▼ │ │ • 容量规划 • 监控告警 • 问题诊断 │ │ • 配置管理 • 备份恢复 • 故障排查 │ │ • 安全加固 • 数据保留 • 紧急恢复 │ │ • 拓扑设计 • 扩缩容 • 性能调优 │ │ │ └─────────────────────────────────────────────────────────────────────────┘  规划与部署 #  在集群上线前完成的关键决策。"
---


# 运维手册

面向生产环境的完整运维指南，帮助您管理和维护 Easysearch 集群的稳定性、安全性和性能。

---

## 运维全景图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Easysearch 运维体系                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                │
│   │   规划部署   │───▶│   日常运维   │───▶│   故障处理   │                │
│   └─────────────┘    └─────────────┘    └─────────────┘                │
│         │                  │                  │                         │
│         ▼                  ▼                  ▼                         │
│   • 容量规划           • 监控告警           • 问题诊断                   │
│   • 配置管理           • 备份恢复           • 故障排查                   │
│   • 安全加固           • 数据保留           • 紧急恢复                   │
│   • 拓扑设计           • 扩缩容             • 性能调优                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 规划与部署

在集群上线前完成的关键决策。

| 主题 | 说明 | 关键操作 |
|------|------|----------|
| [容量规划]({{< relref "/docs/best-practices/index-design.md" >}}) | 硬件选型、分片设计、节点角色规划 | 压测验证、资源估算 |
| [配置管理]({{< relref "./configuration" >}}) | easysearch.yml 核心参数、JVM 调优 | 集群命名、路径规划、堆内存 |
| [部署与恢复]({{< relref "/docs/deployment/install-guide/production-env.md" >}}) | 部署拓扑、滚动升级、灾备方案 | 多 AZ 部署、滚动重启 |
| [安全配置]({{< relref "./security" >}}) | 认证授权、TLS 加密、安全加固 | 启用安全模块、配置角色 |

---

## 日常运维

集群上线后的日常管理任务。

| 主题 | 说明 | 关键操作 |
|------|------|----------|
| [监控]({{< relref "./monitoring" >}}) | 关键指标、健康检查、可视化工具 | `_cluster/health`、`_nodes/stats` |
| [告警]({{< relref "./alerting" >}}) | 告警规则、通知渠道、告警处理 | INFINI Console 告警管理 |
| [数据保留]({{< relref "/docs/features/data-retention/lifecycle" >}}) | 索引生命周期、冷热分层、自动清理 | ILM 策略、Rollup |
| [扩缩容]({{< relref "/docs/operations/cluster-admin/cluster" >}}) | 水平扩展、分片迁移、节点管理 | 添加节点、分片重分配 |

> **备份还原**：请查看 [数据保留与生命周期]({{< relref "/docs/features/data-retention/" >}}) 功能手册的 [备份还原指南]({{< relref "/docs/features/data-retention/backup-restore" >}})

---

## 故障处理

问题发生时的诊断与恢复。

| 主题 | 说明 | 典型场景 |
|------|------|----------|
| [故障排查]({{< relref "./troubleshooting" >}}) | 诊断路径、日志分析、常见问题 | 集群变红、性能下降、写入拒绝 |

---

## 集群与索引管理 API

完整的 API 参考，用于集群和索引的高级管理操作。

### 核心 API

| API | 说明 |
|------|------|
| [常用 API]({{< relref "./cluster-admin/common" >}}) | 高频使用的集群与索引管理 API |
| [集群 API]({{< relref "./cluster-admin/cluster" >}}) | 集群健康、节点信息、分片分配等 |
| [Cat API]({{< relref "./cluster-admin/cat" >}}) | 以表格格式查看集群状态 |

### 索引管理

| API | 说明 |
|------|------|
| [索引增删改查]({{< relref "/docs/operations/data-management/index-management" >}}) | 创建、查看、删除索引，更新映射 |
| [索引设置]({{< relref "/docs/operations/data-management/index-settings" >}}) | 分片、副本、刷新、合并等设置 |
| [索引模板]({{< relref "/docs/operations/data-management/index-templates" >}}) | 创建和管理索引模板 |
| [别名]({{< relref "/docs/operations/data-management/aliases" >}}) | 虚拟索引名、零停机切换 |
| [开关索引与索引限制]({{< relref "/docs/operations/data-management/open-close-index" >}}) | Open/Close、读写控制 |
| [克隆/缩小/拆分]({{< relref "/docs/operations/data-management/clone-shrink-split" >}}) | Clone、Shrink、Split |
| [Rollover]({{< relref "/docs/operations/data-management/index-rollover" >}}) | 索引滚动 |
| [Refresh/Flush/Force Merge]({{< relref "/docs/operations/data-management/refresh-flush-forcemerge" >}}) | 索引维护操作 |
| [索引统计与监控]({{< relref "/docs/operations/data-management/index-stats" >}}) | Stats、Segments、Recovery |
| [数据流]({{< relref "/docs/operations/data-management/data-streams" >}}) | Data Stream 管理 |
| [Reindex]({{< relref "/docs/operations/data-management/reindex" >}}) | 数据重建索引 |

### 生命周期与保留

| API | 说明 |
|------|------|
| [索引生命周期管理]({{< relref "/docs/features/data-retention/ilm.md" >}}) | ILM 策略配置与管理 |
| [备份与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}}) | 快照仓库、备份创建与数据恢复 |
| [快照生命周期管理]({{< relref "/docs/features/data-retention/slm.md" >}}) | SLM 自动快照策略 |
| [时间序列索引优化]({{< relref "/docs/features/data-retention/time-series" >}}) | TimeRangeMergePolicy 合并策略 |

### 高级功能

| API | 说明 |
|------|------|
| [跨集群复制 API]({{< relref "./cluster-admin/ccr" >}}) | CCR 跨集群数据同步 |
| [Rollup]({{< relref "/docs/features/data-retention/rollup" >}}) | 时序数据汇总与降采样 |
| [可搜索快照]({{< relref "/docs/features/data-retention/searchable-snapshot" >}}) | 直接搜索快照中的数据 |
| [任务 API]({{< relref "./cluster-admin/tasks" >}}) | 查看和管理集群任务 |

### 其他

| API | 说明 |
|------|------|
| [节流控制]({{< relref "./cluster-admin/throttling" >}}) | 写入与恢复速率限制 |
| [日志 API]({{< relref "./logs" >}}) | 集群日志查看与配置 |

---

## 快速诊断命令

日常运维最常用的几个命令：

```bash
# 集群健康状态
GET _cluster/health?pretty

# 节点资源概览
GET _cat/nodes?v&h=name,heap.percent,ram.percent,cpu,load_1m,disk.used_percent

# 分片分布
GET _cat/shards?v&s=state

# 未分配分片原因
GET _cluster/allocation/explain

# 热点线程
GET _nodes/hot_threads

# 慢查询日志配置
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s"
}
```

---

## 运维检查清单

### 上线前检查

- [ ] 集群名称已自定义（非默认 `easysearch`）
- [ ] 数据目录独立于安装目录
- [ ] 堆内存设置为可用内存的 50%，且不超过 32GB
- [ ] 禁用 swap 或设置 `bootstrap.memory_lock: true`
- [ ] 安全模块已启用，管理账号已配置
- [ ] TLS 加密已开启（内部通信 + 客户端通信）
- [ ] 快照仓库已配置，备份策略已就绪

### 日常巡检

- [ ] `_cluster/health` 为 green
- [ ] 无长期未分配分片
- [ ] 节点磁盘使用率 < 85%
- [ ] JVM 堆使用率 < 75%
- [ ] 无异常慢查询
- [ ] 备份任务正常执行

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [索引管理]({{< relref "/docs/operations/data-management/_index.md" >}}) | 索引的增删改查、设置、模板、别名、开关、克隆、维护 |
| [Ingest Pipeline]({{< relref "/docs/features/ingest-pipelines/_index.md" >}}) | 数据写入预处理 |

## 最佳实践

- [索引与分片设计]({{< relref "/docs/best-practices/index-design.md" >}})：容量规划、分片策略、性能优化
- [数据生命周期与保留策略]({{< relref "/docs/best-practices/data-lifecycle.md" >}})：ILM 自动化、冷热分层、数据保留
- [查询调优与慢查询排查]({{< relref "/docs/best-practices/query-tuning.md" >}})：性能诊断、瓶颈分析
- [安全与多租户最佳实践]({{< relref "/docs/best-practices/security-and-multi-tenancy.md" >}})：权限设计、数据隔离

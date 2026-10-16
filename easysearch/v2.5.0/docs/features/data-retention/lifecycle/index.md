---
title: "数据生命周期"
date: 0001-01-01
description: "时序数据的保留与淘汰策略：热温冷分层、ILM 自动化、快照归档、数据汇总。"
summary: "数据保留与生命周期管理 #  时间序列数据（日志/指标/审计）有个现实：越新的越值钱，越老的越偶尔被翻。当数据规模持续增长，你面临三个经典问题：
 性能与成本的平衡：最近数据要快速可查，历史数据如何低成本保留？ 合规与可追溯：历史数据要留多久？ 自动化运维：如何减少手工干预？  本页给你一套完整的数据保留策略体系。建议先了解 时间序列数据建模。
 快速决策树 #  开始 → 数据还在写入? ├─ 是 → 放在&#34;热&#34;节点，使用好硬件 │ ├─ 准备自动化? → 用 ILM（推荐） │ └─ 手动管理? → 按时间切索引 │ └─ 不再写入 → 迁移到&#34;温/冷&#34;节点 ├─ 还会经常查询? → 温节点，forcemerge 优化 │ ├─ 很少查询? → 冷节点或关闭索引 │ └─ 长期存档? ├─ 快照 → 删索引（完全清除） └─ 或用 Rollup 汇总压缩  核心策略详解 #  1. 按时间切索引：最简单的删除策略 #  时间序列数据治理的第一优先：按时间切索引。
这样做的好处：
 删除旧数据变成&quot;删整个索引&quot;，而不是&quot;给 N 个文档标记删除&quot; 删索引是异步的、无锁的，不阻塞查询和写入 可以精确控制数据保留窗口  示例：按天切索引"
---


# 数据保留与生命周期管理

时间序列数据（日志/指标/审计）有个现实：**越新的越值钱，越老的越偶尔被翻**。当数据规模持续增长，你面临三个经典问题：

1. **性能与成本的平衡**：最近数据要快速可查，历史数据如何低成本保留？
2. **合规与可追溯**：历史数据要留多久？
3. **自动化运维**：如何减少手工干预？

本页给你一套完整的数据保留策略体系。建议先了解 [时间序列数据建模]({{< relref "/docs/best-practices/data-modeling/time-series" >}})。

---

## 快速决策树

```
开始 → 数据还在写入?
  ├─ 是 → 放在"热"节点，使用好硬件
  │         ├─ 准备自动化? → 用 ILM（推荐）
  │         └─ 手动管理? → 按时间切索引
  │
  └─ 不再写入 → 迁移到"温/冷"节点
      ├─ 还会经常查询? → 温节点，forcemerge 优化
      │
      ├─ 很少查询? → 冷节点或关闭索引
      │
      └─ 长期存档? 
          ├─ 快照 → 删索引（完全清除）
          └─ 或用 Rollup 汇总压缩
```

---

## 核心策略详解

### 1. 按时间切索引：最简单的删除策略

时间序列数据治理的第一优先：**按时间切索引**。

这样做的好处：

- 删除旧数据变成"删整个索引"，而不是"给 N 个文档标记删除"
- 删索引是异步的、无锁的，不阻塞查询和写入
- 可以精确控制数据保留窗口

示例：按天切索引

```
logs-2026-02-20  # 今天的索引，持续写入
logs-2026-02-19  # 昨天，不再写入
logs-2026-02-01  # 1 个月前，可以删除
```

删除过期索引：

```json
DELETE /logs-2026-01*
```

### 2. 热温冷分层：资源利用的金字塔

按照访问频率和时间，将索引分配到不同硬件上：

```
┌─ HOT（热）  → 最好的 SSD、最多内存缓存 → 最近 1-7 天
├─ WARM（温）→ 次级 SSD 或 HDD        → 1 周-1 个月
└─ COLD（冷）→ 廉价存储、最少计算    → 1 个月以上
```

**手动分层（使用节点标签）**

为节点打标签：

```yaml
# easysearch.yml
node.attr.box_type: hot   # 节点类型标记
```

为索引指定位置：

```json
# 热索引：部署到 hot 节点
PUT /logs-2026-02-20/_settings
{
  "index.routing.allocation.include.box_type": "hot"
}

# 7 天后：迁移到 warm 节点
POST /logs-2026-02-13/_settings
{
  "index.routing.allocation.include.box_type": "warm"
}
```

### 3. 索引优化：forcemerge

对不再写入的索引，合并段可以减少文件数量、降低资源占用。

**使用场景**：
- 日索引在第二天就不再写入，此时可以 forcemerge
- 必须先迁移到非热节点，避免影响热数据写入

```json
# 合并为单个段
POST /logs-2026-02-19/_forcemerge?max_num_segments=1
```

> 注意：forcemerge 是 CPU 和 IO 密集操作，应在维护窗口执行，或确保 WARM/COLD 节点有充足资源。

### 4. 关闭索引：保留但休眠

当索引"基本不查但还要保留"（如合规要求保存 1 年），可以关闭它以释放运行时资源。

```json
# 先 flush，再关闭
POST /logs-2025-01*/_flush
POST /logs-2025-01*/_close
```

关闭后的索引：
- ✅ 占用磁盘空间
- ❌ 不占用 JVM 堆、内存缓存、打开的文件句柄
- 需要查询时可快速重新打开：`POST /logs-2025-01*/_open`

### 5. 数据汇总：Rollup

对于指标/监控数据，用 **Rollup** 将细粒度数据聚合为粗粒度数据：

```json
PUT _rollup/jobs/metrics-daily
{
  "rollup": {
    "source_index": "metrics-*",
    "target_index": "rollup_metrics",
    "continuous": true,
    "schedule": {
      "interval": {
        "period": 1,
        "unit": "Days",
        "start_time": 1
      }
    },
    "page_size": 1000,
    "dimensions": [
      {
        "date_histogram": {
          "source_field": "@timestamp",
          "fixed_interval": "1h"
        }
      },
      {
        "terms": {
          "source_field": "host.keyword"
        }
      }
    ],
    "metrics": [
      {
        "source_field": "cpu_usage",
        "metrics": [{ "avg": {} }, { "max": {} }]
      },
      {
        "source_field": "memory_usage",
        "metrics": [{ "avg": {} }, { "max": {} }]
      }
    ]
  }
}
```

优势：
- 原始 1s 精度数据保留 7 天
- 汇总为 1h 精度的数据保留 1 年
- 存储减少 99%，趋势数据完整保留

### 6. 快照归档：长期存储

对于必须存档但不需频繁访问的数据（如审计日志），快照是最佳方案：

```json
# 1. 创建快照仓库
PUT /_snapshot/archive
{
  "type": "s3",
  "settings": {
    "bucket": "my-archive-bucket",
    "region": "us-west-2"
  }
}

# 2. 快照旧索引
PUT /_snapshot/archive/logs-202501-snapshot
{
  "indices": "logs-202501*"
}

# 3. 验证快照完成后，删除原索引
DELETE /logs-202501*
```

> 完整用法见 [备份与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}})

---

## 自动化：ILM 与 SLM

### 索引生命周期管理（ILM）

ILM 可以自动执行上述所有策略：定义一个生命周期策略后，所有匹配的索引自动经历 hot → warm → cold → delete 的阶段。

定义策略：

```json
PUT _ilm/policy/logs-retention-policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0m",
        "actions": {
          "rollover": { "max_docs": 50000000 }  # 5000万文档滚动
        }
      },
      "warm": {
        "min_age": "3d",
        "actions": {
          "set_priority": { "priority": 50 },
          "forcemerge": { "max_num_segments": 1 }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "set_priority": { "priority": 0 },
          "read_only": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

通过模板关联策略，新索引自动管理：

```json
PUT _template/logs-template
{
  "index_patterns": ["logs-*"],
  "settings": {
    "index.lifecycle.name": "logs-retention-policy",
    "index.lifecycle.rollover_alias": "logs"
  },
  "mappings": {
    "properties": {
      "@timestamp": { "type": "date" }
    }
  }
}
```

这样，每个新建的 `logs-*` 索引自动进入生命周期管理，无需手工干预。

> 详细参考：[索引生命周期管理 API]({{< relref "/docs/features/data-retention/ilm.md" >}})

### 快照生命周期管理（SLM）

SLM 自动执行快照，配合 ILM 使用：

```json
PUT _slm/policy/archive-snapshots
{
  "description": "每日快照归档",
  "creation": {
    "schedule": {
      "cron": {
        "expression": "0 0 2 * * ?",
        "timezone": "Asia/Shanghai"
      }
    }
  },
  "snapshot_config": {
    "date_expression": "<snapshot-{now/d}>",
    "repository": "archive",
    "indices": "logs-*"
  },
  "deletion": {
    "condition": {
      "max_age": "365d",
      "min_count": 7,
      "max_count": 50
    }
  }
}
```

---

## 完整案例：日志系统的生命周期

假设日志系统需求：

- 最近 7 天：快速查询，用热节点
- 7-30 天：偶尔查询，用温节点
- 30-90 天：极少查询，用冷节点  
- 90+ 天：归档到 S3，集群删除

**配置步骤：**

1. **创建 ILM 策略**

```json
PUT _ilm/policy/logs-lifecycle
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0m",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "30GB"
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "set_priority": { "priority": 50 },
          "forcemerge": { "max_num_segments": 1 },
          "allocation": { "include": { "box_type": "warm" } }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "set_priority": { "priority": 0 },
          "allocation": { "include": { "box_type": "cold" } }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

2. **创建索引模板**

```json
PUT _template/logs-template
{
  "index_patterns": ["logs-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index.lifecycle.name": "logs-lifecycle",
    "index.lifecycle.rollover_alias": "logs"
  }
}
```

3. **创建初始索引并写入数据**

```json
# 写入时使用别名，不直接写索引
POST /logs/_doc
{
  "@timestamp": "2026-02-20T10:00:00Z",
  "level": "INFO",
  "message": "Application started"
}
```

之后，所有事情自动发生：
- 新索引自动进入 hot 阶段，分配到 hot 节点
- 7 天后自动迁移到 warm 节点并 forcemerge
- 30 天后迁移到 cold 节点
- 90 天后自动删除

---

## 最佳实践总结

| 场景 | 推荐策略 | 优势 |
|------|--------|------|
| **新系统，想简单** | 按时间切 + 手工删除 | 无需复杂配置，易理解 |
| **中型集群，多节点** | 热温冷分层 + ILM | 成本最优，自动化 |
| **大型集群** | ILM + SLM + Rollup | 完全自动化，存储最优 |
| **审计/合规** | 快照 + 归档 | 长期存储，成本低 |
| **指标数据** | Rollup + 短期存储 | 保留趋势，极度省空间 |

**关键建议：**

1. ✅ **优先用时间切索引**，不要依赖按文档删除
2. ✅ **使用 ILM 自动化**，减少人工运维
3. ✅ **热温冷分层**，充分利用不同硬件
4. ✅ **定期备份**，用 SLM 自动化快照
5. ✅ **监控磁盘和 JVM**，提前预警

---

## 相关文档

- [快照与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}})
- [索引生命周期管理 API]({{< relref "/docs/features/data-retention/ilm.md" >}})
- [快照生命周期管理 API]({{< relref "/docs/features/data-retention/slm.md" >}})
- [Rollup 数据汇总]({{< relref "/docs/features/data-retention/rollup" >}})
- [可搜索快照]({{< relref "/docs/features/data-retention/searchable-snapshot" >}})
- [写入限流]({{< relref "/docs/operations/cluster-admin/throttling" >}})
- [时间序列索引优化]({{< relref "/docs/features/data-retention/time-series" >}})



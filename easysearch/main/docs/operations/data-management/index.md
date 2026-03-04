---
title: "索引管理"
date: 0001-01-01
description: "索引的增删改查、设置、映射、别名、开关、克隆、模板、压缩、数据流与重建索引。"
summary: "索引管理 #  涵盖索引生命周期的全部操作：创建、查看、修改、删除，以及设置、映射、别名、模板、压缩、开关、克隆、滚动、维护和监控。
 索引 CRUD #     主题 说明      索引的增删改查 创建/查看/删除索引、更新映射、重建索引、零停机切换    索引设置 分片数、副本数、刷新间隔、段合并、事务日志等    索引模板 预定义索引的 Mapping 和 Settings，自动匹配新索引    别名（Aliases） 虚拟索引名、零停机切换、多索引聚合、过滤别名    索引操作 #     主题 说明      开关索引与索引限制 关闭/打开索引、Index Block 读写控制    克隆/缩小/拆分 Clone、Shrink、Split 调整分片结构    索引滚动（Rollover） 别名或数据流达到阈值时自动滚动到新索引    索引维护 #     主题 说明      Refresh/Flush/Force Merge 手动刷新、持久化、段合并与缓存清理    索引统计与监控 Stats、Segments、Recovery、Shard Stores    索引压缩 ZSTD 压缩、source_reuse 优化    数据组织与迁移 #     主题 说明      数据流 时序数据的自动滚动索引管理    重建索引 使用 Reindex 进行数据迁移与索引重建    数据保留 #     主题 说明      数据生命周期 热温冷分层、ILM 自动化、快照归档    索引生命周期管理（ILM） ILM 策略配置与管理 API    备份与恢复 快照仓库配置与数据恢复    快照生命周期管理（SLM） SLM 自动快照策略    Rollup 数据汇总 使用 Rollup 对时序数据降采样    可搜索快照 直接搜索快照中的数据    时序索引优化 TimeRangeMergePolicy 合并策略    "
---


# 索引管理

涵盖索引生命周期的全部操作：创建、查看、修改、删除，以及设置、映射、别名、模板、压缩、开关、克隆、滚动、维护和监控。

---

## 索引 CRUD

| 主题 | 说明 |
|------|------|
| [索引的增删改查]({{< relref "./index-management" >}}) | 创建/查看/删除索引、更新映射、重建索引、零停机切换 |
| [索引设置]({{< relref "./index-settings" >}}) | 分片数、副本数、刷新间隔、段合并、事务日志等 |
| [索引模板]({{< relref "./index-templates" >}}) | 预定义索引的 Mapping 和 Settings，自动匹配新索引 |
| [别名（Aliases）]({{< relref "./aliases" >}}) | 虚拟索引名、零停机切换、多索引聚合、过滤别名 |

## 索引操作

| 主题 | 说明 |
|------|------|
| [开关索引与索引限制]({{< relref "./open-close-index" >}}) | 关闭/打开索引、Index Block 读写控制 |
| [克隆/缩小/拆分]({{< relref "./clone-shrink-split" >}}) | Clone、Shrink、Split 调整分片结构 |
| [索引滚动（Rollover）]({{< relref "./index-rollover" >}}) | 别名或数据流达到阈值时自动滚动到新索引 |

## 索引维护

| 主题 | 说明 |
|------|------|
| [Refresh/Flush/Force Merge]({{< relref "./refresh-flush-forcemerge" >}}) | 手动刷新、持久化、段合并与缓存清理 |
| [索引统计与监控]({{< relref "./index-stats" >}}) | Stats、Segments、Recovery、Shard Stores |
| [索引压缩]({{< relref "./index-compression" >}}) | ZSTD 压缩、source_reuse 优化 |

## 数据组织与迁移

| 主题 | 说明 |
|------|------|
| [数据流]({{< relref "./data-streams" >}}) | 时序数据的自动滚动索引管理 |
| [重建索引]({{< relref "./reindex" >}}) | 使用 Reindex 进行数据迁移与索引重建 |

## 数据保留

| 主题 | 说明 |
|------|------|
| [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}}) | 热温冷分层、ILM 自动化、快照归档 |
| [索引生命周期管理（ILM）]({{< relref "/docs/features/data-retention/ilm" >}}) | ILM 策略配置与管理 API |
| [备份与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}}) | 快照仓库配置与数据恢复 |
| [快照生命周期管理（SLM）]({{< relref "/docs/features/data-retention/slm" >}}) | SLM 自动快照策略 |
| [Rollup 数据汇总]({{< relref "/docs/features/data-retention/rollup" >}}) | 使用 Rollup 对时序数据降采样 |
| [可搜索快照]({{< relref "/docs/features/data-retention/searchable-snapshot" >}}) | 直接搜索快照中的数据 |
| [时序索引优化]({{< relref "/docs/features/data-retention/time-series" >}}) | TimeRangeMergePolicy 合并策略 |

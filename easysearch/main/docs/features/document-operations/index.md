---
title: "文档操作"
date: 0001-01-01
description: "文档的 CRUD（写入、检索、更新、删除）、批量操作、按查询更新/删除，以及写入链路上的预处理。"
summary: "Easysearch 提供一组完整的文档级别 RESTful API，覆盖单文档 CRUD、批量操作与辅助查询。
单文档操作 #     API 说明      Index API 写入文档（创建或覆盖），支持自动生成 ID 和仅创建模式    Get API 按 ID 检索文档，支持 _source 过滤、HEAD 存在性检查    Update API 部分更新文档，支持 doc 合并、脚本更新、Upsert    Delete API 按 ID 删除单条文档    批量操作 #     API 说明      Bulk API 在单个请求中执行多个 index / create / update / delete    Multi Get API 一次请求批量获取多条文档    按查询操作 #     API 说明      Update by Query 按查询条件批量更新文档，支持脚本、异步、流量控制    Delete by Query 按查询条件批量删除文档，支持异步、流量控制    辅助 API #     API 说明      Count API 返回匹配查询的文档数量    Term Vectors API 获取字段的词项信息（词频、位置、偏移量）    写入链路 #     主题 说明      摄取管道（Ingest Pipelines） 写入前的预处理流水线：Processors、管道 API、故障处理    推荐阅读顺序 #    Index API：写入文档  Get API：检索文档  Update API：部分更新  Delete API：删除文档  Bulk API：批量写入  Multi Get API：批量检索  Update by Query：按条件批量更新  Delete by Query：按条件批量删除  分布式写入过程：请求在分片间的流转  并发控制与版本：乐观锁与安全更新模式  摄取管道：写入链路上的预处理流水线  _source 与字段存储：字段存储策略与成本权衡  相关章节 #     方向 参见     跨索引迁移数据  Reindex   索引管理、模板、别名  索引管理   分片规划与写入性能  索引与分片设计   ILM、冷热分层、自动清理  数据生命周期与保留策略   文档设计与更新模式  数据建模    "
---


Easysearch 提供一组完整的文档级别 RESTful API，覆盖单文档 CRUD、批量操作与辅助查询。

## 单文档操作

| API | 说明 |
|-----|------|
| [Index API]({{< relref "./index-api.md" >}}) | 写入文档（创建或覆盖），支持自动生成 ID 和仅创建模式 |
| [Get API]({{< relref "./get-api.md" >}}) | 按 ID 检索文档，支持 `_source` 过滤、HEAD 存在性检查 |
| [Update API]({{< relref "./update-api.md" >}}) | 部分更新文档，支持 `doc` 合并、脚本更新、Upsert |
| [Delete API]({{< relref "./delete-api.md" >}}) | 按 ID 删除单条文档 |

## 批量操作

| API | 说明 |
|-----|------|
| [Bulk API]({{< relref "./bulk-api.md" >}}) | 在单个请求中执行多个 index / create / update / delete |
| [Multi Get API]({{< relref "./multi-get-api.md" >}}) | 一次请求批量获取多条文档 |

## 按查询操作

| API | 说明 |
|-----|------|
| [Update by Query]({{< relref "./update-by-query.md" >}}) | 按查询条件批量更新文档，支持脚本、异步、流量控制 |
| [Delete by Query]({{< relref "./delete-by-query.md" >}}) | 按查询条件批量删除文档，支持异步、流量控制 |

## 辅助 API

| API | 说明 |
|-----|------|
| [Count API]({{< relref "./count-api.md" >}}) | 返回匹配查询的文档数量 |
| [Term Vectors API]({{< relref "./term-vectors-api.md" >}}) | 获取字段的词项信息（词频、位置、偏移量） |

## 写入链路

| 主题 | 说明 |
|------|------|
| [摄取管道（Ingest Pipelines）]({{< relref "/docs/features/ingest-pipelines/_index.md" >}}) | 写入前的预处理流水线：Processors、管道 API、故障处理 |

## 推荐阅读顺序

1. [Index API]({{< relref "./index-api.md" >}})：写入文档
2. [Get API]({{< relref "./get-api.md" >}})：检索文档
3. [Update API]({{< relref "./update-api.md" >}})：部分更新
4. [Delete API]({{< relref "./delete-api.md" >}})：删除文档
5. [Bulk API]({{< relref "./bulk-api.md" >}})：批量写入
6. [Multi Get API]({{< relref "./multi-get-api.md" >}})：批量检索
7. [Update by Query]({{< relref "./update-by-query.md" >}})：按条件批量更新
8. [Delete by Query]({{< relref "./delete-by-query.md" >}})：按条件批量删除
9. [分布式写入过程]({{< relref "/docs/fundamentals/distributed-write.md" >}})：请求在分片间的流转
10. [并发控制与版本]({{< relref "/docs/fundamentals/concurrency-and-versioning.md" >}})：乐观锁与安全更新模式
11. [摄取管道]({{< relref "/docs/features/ingest-pipelines/_index.md" >}})：写入链路上的预处理流水线
12. [_source 与字段存储]({{< relref "/docs/fundamentals/source-and-stored-fields.md" >}})：字段存储策略与成本权衡

## 相关章节

| 方向 | 参见 |
|------|------|
| 跨索引迁移数据 | [Reindex]({{< relref "/docs/operations/data-management/reindex.md" >}}) |
| 索引管理、模板、别名 | [索引管理]({{< relref "/docs/operations/data-management/_index.md" >}}) |
| 分片规划与写入性能 | [索引与分片设计]({{< relref "/docs/best-practices/index-design.md" >}}) |
| ILM、冷热分层、自动清理 | [数据生命周期与保留策略]({{< relref "/docs/best-practices/data-lifecycle.md" >}}) |
| 文档设计与更新模式 | [数据建模]({{< relref "/docs/best-practices/data-modeling/_index.md" >}}) |

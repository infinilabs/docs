---
title: "功能手册"
date: 0001-01-01
description: "各模块功能说明：文档操作、全文搜索、向量检索、聚合等。"
summary: "功能手册 #  Easysearch 各功能模块的详细说明与使用指南。本手册涵盖了从文档写入、搜索查询、聚合分析到映射配置等核心功能，帮助您充分利用 Easysearch 构建高性能搜索和数据分析应用。
 阅读建议：如果您刚接触 Easysearch，建议先阅读 基础概念 和 快速上手，然后根据需求查阅对应模块。
 写入与文档操作 #    文档操作：写入、更新、删除、批量操作、CRUD  摄取管道：写入前预处理（Pipeline Processors）  搜索 #  搜索能力分为六大维度，从基础的 Query DSL 到向量、语义、多模态等高级搜索。
基础与参考 #    查询 DSL：Query DSL 基础、参数说明、精确查询、复合查询、关联查询、专业查询  全文搜索 #    全文搜索：match、multi_match、短语匹配、多字段搜索、分页排序  相关性调优：评分机制、BM25、boosting、调优策略  Span 查询：文本级精细查询、词项距离、位置控制  向量与语义 #    向量搜索：k-NN 索引、高维向量存储与检索  语义搜索：Embedding 向量匹配、Hybrid 检索  多模态搜索：图片、音视频跨模态检索  地理搜索 #    地理搜索：geo_point、geo_shape、地理聚合、距离查询  搜索增强 #    搜索管道：搜索前后处理、查询改写（Search Pipelines）  聚合与分析 #    聚合分析：指标聚合、桶聚合、管道聚合  Mapping 与文本分析 #    Mapping：字段映射、字段类型、动态映射  文本分析：分析器、分词器、同义词、停用词  SQL 与规则引擎 #    SQL 查询：SQL 接口与 JDBC  规则引擎：写入时实时规则匹配与自动打标  集群与数据管理 #    异步搜索：大查询后台执行，非阻塞  跨集群搜索：一次查询覆盖多个集群  跨集群复制：数据跨集群同步与容灾  快照搜索：直接在备份数据上搜索  Rollup 数据汇总：时序数据压缩聚合  写入限流：节点/索引/分片级写入控制  安全与合规 #    国密与国产化：SM2/SM3/SM4、国产 CPU/OS 适配  功能速查 #     需求 功能模块     写入、更新、删除文档  文档操作   按关键词精确匹配  精确查询   中文全文搜索  全文搜索 + IK 分词器   按时间、数值分组统计  桶聚合   计算均值、求和等指标  指标聚合   地理位置范围搜索  地理搜索   向量相似度检索  向量搜索   自然语言语义搜索  语义搜索   以图搜图 / 跨模态检索  多模态搜索   写入时自动添加/转换字段  摄取管道   用 SQL 语法查询  SQL 查询   写入时自动打标与规则匹配  规则引擎   跨多个集群统一查询  跨集群搜索   数据异地容灾同步  跨集群复制   大查询后台执行  异步搜索   查询备份/冷数据  快照搜索   时序数据压缩与长期保留  Rollup 数据汇总   控制写入速度保护查询  写入限流   国密算法与国产化部署  国密与国产化    API 参考 #  各模块的详细 API 说明已整合至对应章节（搜索 API、聚合 API、文档 API、摄取管道 API、SQL 等）。其它参考见 参考手册。"
---


# 功能手册

Easysearch 各功能模块的详细说明与使用指南。本手册涵盖了从文档写入、搜索查询、聚合分析到映射配置等核心功能，帮助您充分利用 Easysearch 构建高性能搜索和数据分析应用。

> **阅读建议**：如果您刚接触 Easysearch，建议先阅读 [基础概念]({{< relref "../fundamentals/" >}}) 和 [快速上手]({{< relref "../quick-start/" >}})，然后根据需求查阅对应模块。

## 写入与文档操作

- **[文档操作]({{< relref "./document-operations/" >}})**：写入、更新、删除、批量操作、CRUD
- **[摄取管道]({{< relref "./ingest-pipelines/_index.md" >}})**：写入前预处理（Pipeline Processors）

## 搜索

搜索能力分为六大维度，从基础的 Query DSL 到向量、语义、多模态等高级搜索。

### 基础与参考
- **[查询 DSL]({{< relref "./query-dsl/" >}})**：Query DSL 基础、参数说明、精确查询、复合查询、关联查询、专业查询

### 全文搜索
- **[全文搜索]({{< relref "./fulltext-search/" >}})**：match、multi_match、短语匹配、多字段搜索、分页排序
- **[相关性调优]({{< relref "./fulltext-search/relevance/" >}})**：评分机制、BM25、boosting、调优策略
- **[Span 查询]({{< relref "./query-dsl/span/" >}})**：文本级精细查询、词项距离、位置控制

### 向量与语义
- **[向量搜索]({{< relref "./vector-search/_index.md" >}})**：k-NN 索引、高维向量存储与检索
- **[语义搜索]({{< relref "./semantic-search/_index.md" >}})**：Embedding 向量匹配、Hybrid 检索
- **[多模态搜索]({{< relref "./multimodal-search/_index.md" >}})**：图片、音视频跨模态检索

### 地理搜索
- **[地理搜索]({{< relref "./geo-search/" >}})**：geo_point、geo_shape、地理聚合、距离查询

### 搜索增强
- **[搜索管道]({{< relref "./query-dsl/search-pipelines/" >}})**：搜索前后处理、查询改写（Search Pipelines）

## 聚合与分析

- **[聚合分析]({{< relref "./aggregations/" >}})**：指标聚合、桶聚合、管道聚合

## Mapping 与文本分析

- **[Mapping]({{< relref "./mapping-and-analysis/" >}})**：字段映射、字段类型、动态映射
- **[文本分析]({{< relref "./mapping-and-analysis/text-analysis/" >}})**：分析器、分词器、同义词、停用词

## SQL 与规则引擎

- **[SQL 查询]({{< relref "./sql/" >}})**：SQL 接口与 JDBC
- **[规则引擎]({{< relref "./rule-engine" >}})**：写入时实时规则匹配与自动打标

## 集群与数据管理

- **[异步搜索]({{< relref "./async-search" >}})**：大查询后台执行，非阻塞
- **[跨集群搜索]({{< relref "./cross-cluster-search" >}})**：一次查询覆盖多个集群
- **[跨集群复制]({{< relref "/docs/operations/cluster-admin/ccr" >}})**：数据跨集群同步与容灾
- **[快照搜索]({{< relref "./data-retention/searchable-snapshot" >}})**：直接在备份数据上搜索
- **[Rollup 数据汇总]({{< relref "./data-retention/rollup" >}})**：时序数据压缩聚合
- **[写入限流]({{< relref "/docs/operations/cluster-admin/throttling" >}})**：节点/索引/分片级写入控制

## 安全与合规

- **[国密与国产化]({{< relref "./national-encryption" >}})**：SM2/SM3/SM4、国产 CPU/OS 适配

## 功能速查

| 需求                          | 功能模块                                                               |
| ----------------------------- | ---------------------------------------------------------------------- |
| 写入、更新、删除文档          | [文档操作]({{< relref "./document-operations/" >}})                    |
| 按关键词精确匹配              | [精确查询]({{< relref "./query-dsl/term-based-query/" >}})             |
| 中文全文搜索                  | [全文搜索]({{< relref "./fulltext-search/" >}}) + IK 分词器            |
| 按时间、数值分组统计          | [桶聚合]({{< relref "./aggregations/bucket-aggregations/" >}}) |
| 计算均值、求和等指标          | [指标聚合]({{< relref "./aggregations/metric-aggregations/" >}}) |
| 地理位置范围搜索              | [地理搜索]({{< relref "./geo-search/" >}})                             |
| 向量相似度检索                | [向量搜索]({{< relref "./vector-search/_index.md" >}})                 |
| 自然语言语义搜索              | [语义搜索]({{< relref "./semantic-search/_index.md" >}})               |
| 以图搜图 / 跨模态检索         | [多模态搜索]({{< relref "./multimodal-search/_index.md" >}})           |
| 写入时自动添加/转换字段       | [摄取管道]({{< relref "./ingest-pipelines/_index.md" >}})    |
| 用 SQL 语法查询               | [SQL 查询]({{< relref "./sql/" >}})                                    |
| 写入时自动打标与规则匹配      | [规则引擎]({{< relref "./rule-engine" >}})                             |
| 跨多个集群统一查询            | [跨集群搜索]({{< relref "./cross-cluster-search" >}})                  |
| 数据异地容灾同步              | [跨集群复制]({{< relref "/docs/operations/cluster-admin/ccr" >}})             |
| 大查询后台执行                | [异步搜索]({{< relref "./async-search" >}})                            |
| 查询备份/冷数据               | [快照搜索]({{< relref "./data-retention/searchable-snapshot" >}})      |
| 时序数据压缩与长期保留        | [Rollup 数据汇总]({{< relref "./data-retention/rollup" >}})            |
| 控制写入速度保护查询          | [写入限流]({{< relref "/docs/operations/cluster-admin/throttling" >}})                        |
| 国密算法与国产化部署          | [国密与国产化]({{< relref "./national-encryption" >}})                 |

## API 参考

各模块的详细 API 说明已整合至对应章节（搜索 API、聚合 API、文档 API、摄取管道 API、SQL 等）。其它参考见 [参考手册]({{< relref "/docs/api-reference/" >}})。

---
title: "基础理论"
date: 0001-01-01
description: "背后的理论知识：核心概念、Lucene 原理、分布式架构、写入机制等。"
summary: "基础理论 #  理解 Easysearch 背后的核心原理，建立正确的心智模型。
核心概念 #    核心概念：索引、文档、分片、副本、集群、节点  Lucene 底层原理：倒排索引、段结构、文件格式、BM25 评分模型  分布式原理 #    分布式基础：集群扩容、分片分配、故障转移  分布式写入过程：文档路由（Murmur3 哈希）、主副本交互、mget/bulk 执行路径  分布式搜索执行过程：query/fetch 两阶段、深分页、游标查询  写入与索引 #    写入与存储机制：近实时搜索、refresh、translog、flush、段合并  文档建模：字段设计、范式取舍、标识符选择  并发控制与版本管理：乐观并发、seq_no/primary_term、外部版本  _source 与存储字段：_source、doc_values、stored fields 的区别与取舍  搜索与分析 #    映射与分析入门：精确值 VS 全文、分析器、映射类型  RESTful 与 Query DSL：API 风格与查询语法  相关性与排序：BM25 评分、排序、explain 调试  NLP 自然语言处理：分词、同义词、语义搜索  聚合与统计分析 #    聚合与数据分析：聚合基础、桶聚合、指标聚合、管道聚合的完整教程  "
---


# 基础理论

理解 Easysearch 背后的核心原理，建立正确的心智模型。

## 核心概念

- **[核心概念]({{< relref "./concepts.md" >}})**：索引、文档、分片、副本、集群、节点
- **[Lucene 底层原理]({{< relref "./lucene-internals.md" >}})**：倒排索引、段结构、文件格式、BM25 评分模型

## 分布式原理

- **[分布式基础]({{< relref "./distributed.md" >}})**：集群扩容、分片分配、故障转移
- **[分布式写入过程]({{< relref "./distributed-write.md" >}})**：文档路由（Murmur3 哈希）、主副本交互、mget/bulk 执行路径
- **[分布式搜索执行过程]({{< relref "./distributed-search.md" >}})**：query/fetch 两阶段、深分页、游标查询

## 写入与索引

- **[写入与存储机制]({{< relref "./write-and-storage.md" >}})**：近实时搜索、refresh、translog、flush、段合并
- **[文档建模]({{< relref "./document-model.md" >}})**：字段设计、范式取舍、标识符选择
- **[并发控制与版本管理]({{< relref "./concurrency-and-versioning.md" >}})**：乐观并发、seq_no/primary_term、外部版本
- **[_source 与存储字段]({{< relref "./source-and-stored-fields.md" >}})**：_source、doc_values、stored fields 的区别与取舍

## 搜索与分析

- **[映射与分析入门]({{< relref "./mapping-analysis-intro.md" >}})**：精确值 VS 全文、分析器、映射类型
- **[RESTful 与 Query DSL]({{< relref "./restful-query-dsl.md" >}})**：API 风格与查询语法
- **[相关性与排序]({{< relref "./relevance-and-sorting.md" >}})**：BM25 评分、排序、explain 调试
- **[NLP 自然语言处理]({{< relref "./nlp.md" >}})**：分词、同义词、语义搜索

## 聚合与统计分析

- **[聚合与数据分析]({{< relref "./aggregations-data-analysis.md" >}})**：聚合基础、桶聚合、指标聚合、管道聚合的完整教程

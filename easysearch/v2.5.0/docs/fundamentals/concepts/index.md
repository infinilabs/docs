---
title: "核心概念"
date: 0001-01-01
description: "索引、文档、分片、副本、集群与节点——Easysearch 的基本术语与架构概览。"
summary: "核心概念 #  Easysearch 是一个分布式搜索分析引擎，建立在 Apache Lucene 基础之上。本页介绍 Easysearch 的核心概念，帮助你快速建立正确的心智模型。
什么是 Easysearch #  Easysearch 是一个高性能的分布式搜索引擎，建立在全文搜索引擎库 Apache Lucene 基础之上。Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库。
但 Lucene 仅仅只是一个库。为了充分发挥其功能，你需要使用 Java 并将 Lucene 直接集成到应用程序中。Easysearch 在此基础上提供了一套简单一致的 RESTful API，使全文检索变得简单。
Easysearch 不仅仅是全文搜索引擎，它可以被准确地形容为：
 一个分布式的实时文档存储，每个字段可以被索引与搜索 一个分布式实时分析搜索引擎 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或非结构化数据  面向文档 #  Easysearch 是面向文档（Document-Oriented）的：它存储整个对象或文档，并索引每个文档的内容使之可以被检索。在 Easysearch 中，我们对文档进行索引、检索、排序和过滤——而不是对行列数据。
JSON #  Easysearch 使用 JSON 作为文档的序列化格式：
{ &#34;email&#34;: &#34;john@smith.com&#34;, &#34;first_name&#34;: &#34;John&#34;, &#34;last_name&#34;: &#34;Smith&#34;, &#34;info&#34;: { &#34;bio&#34;: &#34;Eco-warrior and defender of the weak&#34;, &#34;age&#34;: 25, &#34;interests&#34;: [ &#34;dolphins&#34;, &#34;whales&#34; ] }, &#34;join_date&#34;: &#34;2014/05/01&#34; } 在 Easysearch 中将对象转化为 JSON 后构建索引，要比在扁平的表结构中简单得多。"
---


# 核心概念

Easysearch 是一个分布式搜索分析引擎，建立在 Apache Lucene 基础之上。本页介绍 Easysearch 的核心概念，帮助你快速建立正确的心智模型。

## 什么是 Easysearch

Easysearch 是一个高性能的分布式搜索引擎，建立在全文搜索引擎库 Apache Lucene 基础之上。Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库。

但 Lucene 仅仅只是一个库。为了充分发挥其功能，你需要使用 Java 并将 Lucene 直接集成到应用程序中。Easysearch 在此基础上提供了一套简单一致的 RESTful API，使全文检索变得简单。

Easysearch 不仅仅是全文搜索引擎，它可以被准确地形容为：

* 一个分布式的实时文档存储，每个字段可以被索引与搜索
* 一个分布式实时分析搜索引擎
* 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或非结构化数据

## 面向文档

Easysearch 是面向文档（Document-Oriented）的：它存储整个对象或文档，并索引每个文档的内容使之可以被检索。在 Easysearch 中，我们对文档进行索引、检索、排序和过滤——而不是对行列数据。

### JSON

Easysearch 使用 JSON 作为文档的序列化格式：

```json
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "info": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01"
}
```

在 Easysearch 中将对象转化为 JSON 后构建索引，要比在扁平的表结构中简单得多。

## 文档元数据

一个文档不仅包含数据，还包含元数据：

| 元数据 | 说明 |
|--------|------|
| `_index` | 文档所属的索引，是逻辑命名空间 |
| `_id` | 文档的唯一标识符，可自定义或自动生成 |
| `_source` | 文档的原始 JSON 内容 |

`_index` 和 `_id` 组合可以唯一确定 Easysearch 中的一个文档。

> 更多文档建模内容，详见[文档设计]({{< relref "/docs/best-practices/data-modeling/document-design.md" >}})。

## 索引与分片

**索引（Index）** 是逻辑上的数据集合，通常对应一类业务数据（例如 `orders`、`products`）。索引由两类要素决定：

* **settings**：分片数、副本数、刷新间隔、分析器配置等
* **mappings**：字段结构与类型、动态映射规则等

一个索引会被切分为多个**分片（shard）**，分布在不同节点上。分片是索引在物理层面的切分单元，可以理解为"一个独立的 Lucene 索引"。

* **主分片（primary shard）**：负责接收写入并生成变更（默认 **1** 个主分片）
* **副本分片（replica shard）**：用于高可用与提升查询吞吐（默认 **1** 份副本）

分片的数量与大小直接影响：

* 查询并行度与延迟
* 写入并行度与热点
* 资源使用与故障恢复时间

> **"索引"一词的多重含义**
>
> 在 Easysearch 语境中，"索引"有三层含义：
>
> - **索引（名词）**：类似于关系数据库中的"数据库"，是存储文档的逻辑容器
> - **索引（动词）**：将一个文档存储到索引中以便被检索和查询
> - **倒排索引**：Easysearch 底层使用的数据结构，详见 [Lucene 底层原理]({{< relref "./lucene-internals.md" >}})

## 副本

副本是主分片的冗余拷贝：

* 主分片所在节点故障时可提升为主分片，保证服务连续性
* 可以承载查询流量，提升读吞吐

副本数越多，可用性与查询吞吐通常越好，但成本更高（磁盘/CPU/内存/网络）。

## 集群与节点

一个运行中的 Easysearch 实例称为一个**节点**，**集群**是由一个或多个拥有相同 `cluster.name` 配置的节点组成，它们共同承担数据和负载的压力。

- 当有节点加入或移除时，集群会自动重新分配数据
- 被选举为**主节点**的节点负责管理集群范围内的变更（增删索引、增删节点等），但不需要参与文档级别的操作
- 每个节点都知道任意文档所处的位置，能够将请求直接转发到正确的节点

> 集群的扩容、故障转移和分片分配的详细过程，请参阅[分布式基础]({{< relref "./distributed.md" >}})。

## 路由

路由决定一份文档"落到哪个主分片"。默认基于 `_id` 计算哈希并映射到分片号。

- **写入路由**：决定写入去哪个主分片
- **查询路由**：提供与写入一致的路由值，可将查询范围从"全分片"缩小到"单分片/少量分片"

> 路由公式与详细机制，请参阅[分布式写入过程]({{< relref "./distributed-write.md" >}})。

## 概念速查

| 概念 | 说明 |
|------|------|
| **Index** | 逻辑集合，对应一类业务数据 |
| **Shard** | 物理切分单元，每个分片是一个 Lucene 索引 |
| **Replica** | 主分片的冗余拷贝，用于高可用与读扩展 |
| **Document** | 最小数据单元，JSON 格式 |
| **Cluster** | 一组协同工作的节点 |
| **Node** | 一个运行中的 Easysearch 实例 |
| **Routing** | 决定文档归属哪个分片的机制 |

下一步可以继续阅读：

- [分布式基础]({{< relref "./distributed.md" >}})：集群扩容、故障恢复、分片分配
- [Lucene 底层原理]({{< relref "./lucene-internals.md" >}})：倒排索引与段结构
- [数据建模]({{< relref "/docs/best-practices/data-modeling/" >}})：文档设计、Nested、Parent-Child、反范式


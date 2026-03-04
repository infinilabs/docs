---
title: "索引压缩"
date: 0001-01-01
description: "ZSTD 压缩编码、source_reuse 优化，降低存储成本。"
summary: "索引压缩 #  索引编码 #  索引编码决定索引的存储字段如何被压缩和存储在磁盘上。索引编码由静态的 index.codec 设置来控制，该设置指定压缩算法。这个设置会影响索引分片的大小和索引操作的性能。
Easysearch 提供了多种基于索引编码的压缩方案，以降低索引的存储成本。
default – 该编码使用LZ4算法和预设字典，优先考虑性能而非压缩比。与best_compression相比，它提供更快的索引和搜索操作，但可能导致更大的索引/分片大小。如果在索引设置中未提供编码，则默认使用LZ4作为压缩算法。
best_compression – 该编码底层使用zlib算法进行压缩。它能实现高压缩比，从而减小索引大小。然而，这可能会增加索引操作期间的额外CPU使用，并可能随后导致较高的索引和搜索延迟。
从 Easysearch 1.1 开始，增加了基于 Zstandard 压缩算法的新编码方式。这种算法在压缩比和速度之间提供了良好的平衡。
ZSTD 与默认编解码器相比，该编解码器提供了与best_compression编解码器相当的压缩比，CPU使用合理，索引和搜索性能也有所提高。
source 复用 #  source_reuse： 启用 source_reuse 配置项能够去除 _source 字段中与 doc_values 或倒排索引重复的部分，从而有效减小索引总体大小，这个功能对日志类索引效果尤其明显。
source_reuse 支持对以下数据类型进行压缩：keyword，integer，long，short，boolean，float，half_float，double，geo_point，ip， 如果是 text 类型，需要默认启用 keyword 类型的 multi-field 映射。 以上类型必须启用 doc_values 映射（默认启用）才能压缩。
使用限制 #    当索引里包含 nested 类型映射，或插件额外提供的数据类型时，不能启用 source_reuse，例如 knn 索引。
  使用 source_reuse 压缩时，keyword 类型的字段最好不要设置 ignore_above 属性，设置过短的值可能会导致字段内容无法展示。
  压缩效果对比 #  Easysearch 压缩效果对比如下"
---


# 索引压缩

## 索引编码
索引编码决定索引的存储字段如何被压缩和存储在磁盘上。索引编码由静态的 index.codec 设置来控制，该设置指定压缩算法。这个设置会影响索引分片的大小和索引操作的性能。

Easysearch 提供了多种基于索引编码的压缩方案，以降低索引的存储成本。

default – 该编码使用LZ4算法和预设字典，优先考虑性能而非压缩比。与best_compression相比，它提供更快的索引和搜索操作，但可能导致更大的索引/分片大小。如果在索引设置中未提供编码，则默认使用LZ4作为压缩算法。

best_compression – 该编码底层使用zlib算法进行压缩。它能实现高压缩比，从而减小索引大小。然而，这可能会增加索引操作期间的额外CPU使用，并可能随后导致较高的索引和搜索延迟。

从 Easysearch 1.1 开始，增加了基于 Zstandard 压缩算法的新编码方式。这种算法在压缩比和速度之间提供了良好的平衡。

ZSTD 与默认编解码器相比，该编解码器提供了与best_compression编解码器相当的压缩比，CPU使用合理，索引和搜索性能也有所提高。

## source 复用
source_reuse：
启用 source_reuse 配置项能够去除 _source 字段中与 doc_values 或倒排索引重复的部分，从而有效减小索引总体大小，这个功能对日志类索引效果尤其明显。

source_reuse 支持对以下数据类型进行压缩：keyword，integer，long，short，boolean，float，half_float，double，geo_point，ip，
如果是 text 类型，需要默认启用 keyword 类型的 multi-field 映射。
以上类型必须启用 doc_values 映射（默认启用）才能压缩。

### 使用限制
1. 当索引里包含 nested 类型映射，或插件额外提供的数据类型时，不能启用 source_reuse，例如 knn 索引。

2. 使用 source_reuse 压缩时，keyword 类型的字段最好不要设置 ignore_above 属性，设置过短的值可能会导致字段内容无法展示。

## 压缩效果对比
Easysearch 压缩效果对比如下
- 使用 Nginx 日志作为数据样本
- 就 Elasticsearch 6.4.3 和 Easysearch 1.1 进行对比
- 默认未开启压缩
- 开启 best_compression 压缩
- 开启 ZSTD 压缩
- 开启 ZSTD 加 _source 优化

Easysearch 1.1 版本 相比 Elasticsearch 索引整体大小降低了40%~50%。

**Elasticsearch v6.4.3**

| index         | pri | rep | docs.count | store.size | pri.store.size |
|---------------|-----|-----|------------|------------|----------------|
| nginxt_default|  5  |  1  |   1000000  |   413.1mb  |      413.1mb   |
| nginx_best    |  1  |  1  |   1000000  |   316.2mb  |      316.2mb   |

**Easysearch v1.1**

| index         | pri | rep | docs.count | store.size | pri.store.size |
|---------------|-----|-----|------------|------------|----------------|
| nginxt_default|  1  |  1  |   1000000  |   321.6mb  |      321.6mb   |
| nginx_best    |  1  |  1  |   1000000  |   262.8mb  |      262.8mb   |
| nginx_zstd    |  1  |  1  |   1000000  |   205.6mb  |      205.6mb   |
| nginx_reuse   |  1  |  1  |   1000000  |     157mb  |        157mb   |


## 如何启用

### 单个索引配置示例
创建并设置索引的 codec 为ZSTD：
```json
PUT test-index
{
  "settings": {
    "index.codec": "ZSTD"
  }
}
```

创建并设置索引的 codec 为ZSTD，并启用 source_reuse：
```json
PUT test-index
{
  "settings": {
    "index.codec": "ZSTD",
    "index.source_reuse": "true"
  }
}
```

### 索引模板配置示例
在 index_template 配置 ZSTD 和 source_reuse：
```json
PUT _index_template/daily_logs
{
  "index_patterns": [
    "logs-2020-*"
  ],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.codec": "ZSTD",
      "index.source_reuse": true
    },
    "mappings": {
      "properties": {
        "field1": {
          "type": "double"
        }
      }
    }
  }
}
```

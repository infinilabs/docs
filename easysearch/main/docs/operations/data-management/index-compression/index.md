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

### 2.1.0 起新增的 ZSTD 使用方式
2.1.0 起，ZSTD 可以通过 `index.compression.zstd.jni` 请求使用 native/JNI 后端以提升性能：

- `index.codec=ZSTD` 且 `index.compression.zstd.jni=false`：不起用 JNI（默认）
- `index.codec=ZSTD` 且 `index.compression.zstd.jni=true`：优先使用 native/JNI 后端

同时新增以下配置项（仅在启用 JNI 时生效）：
- `index.compression.zstd.level`：压缩级别（默认 `3`）
- `index.compression.zstd.dict`：是否启用字典压缩（默认 `true`，且必须为 `true`）

提示：从 2.1.0 起，`index.codec` 支持大小写不敏感，例如 `zstd` 也可以被识别为 `ZSTD`。

提示：如果运行环境不支持 native ZSTD，例如 Windows、glibc 版本过低、native 库缺失或加载失败，Easysearch 不会因为 ZSTD 直接崩溃：

- 对新建的 `ZSTD` 索引，会在日志中记录 `WARN`
- 如果请求了 `index.compression.zstd.jni=true`，但 native 后端不可用，新索引会自动回退为可兼容的纯 Java ZSTD 写入格式
- 已经写成旧 `Lucene912ZSTDV3` native 格式的索引，在 native 后端不可用的节点上不会再导致进程崩溃，但会以明确错误提示该索引需要 native ZSTD 后端

> 内部说明（**2.3.0 新增**）：从 2.3.0 起，每个 ZSTD 索引在创建时会由系统写入私有设置 `index.compression.zstd.resolved_format`，**把创建时解析出的 ZSTD 格式版本固化到索引元数据**（取值为枚举名 `V1` / `V2` / `V3`，分别对应底层 codec 名 `ZSTD` / `Lucene912ZSTD` / `Lucene912ZSTDV3`）。解析规则：创建版本早于 2.0.0 的旧索引解析为 `V1`；其余索引在 `index.compression.zstd.jni=true` 且运行环境 native 后端支持 V3 时解析为 `V3`，否则解析为 `V2`（纯 Java）。**该设置的一个关键用途是兼容性回退**：当请求了 `index.compression.zstd.jni=true` 但运行环境不支持 native ZSTD（如 Windows、JDK 低于 21、native 库缺失或加载失败）时，系统会在创建时把格式回退为纯 Java 的 `V2` 并固化，使索引始终以兼容的纯 Java 格式写入与读取，避免在无 native 后端的节点上崩溃；反之若创建时 native 可用解析为 `V3` 也会固化，使各节点明确该索引需要 native 后端才能读取。该设置带 `PrivateIndex`、`Final` 属性，**仅在索引创建时由系统写入、此后不可变，无需也不建议手动设置**；后续读取与后台合并时通过它识别格式并选择 codec（仅 `V3` 启用 native 后端并应用 `index.compression.zstd.level`，其余格式使用默认压缩级别）。

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

创建并启用 JNI（2.1.0 起）：
```json
PUT test-index
{
  "settings": {
    "index.codec": "ZSTD",
    "index.compression.zstd.jni": true
  }
}
```

自定义 ZSTD 压缩级别（2.1.0 起）：
```json
PUT test-index
{
  "settings": {
    "index.codec": "ZSTD",
    "index.compression.zstd.jni": true,
    "index.compression.zstd.level": 6
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

## 注意事项
`index.compression.zstd.jni` 是创建期生效的静态设置，已有索引不能在后续通过 `PUT /<index>/_settings` 动态修改。
如果老索引需要启用 JNI，请新建目标索引（设置 `index.compression.zstd.jni=true`）后通过 `reindex` 迁移数据。

补充说明：

- `index.compression.zstd.jni=true` 表示“创建时优先尝试 native ZSTD”，不表示在所有操作系统和运行环境上都一定能启用 native 后端
- 当 native 后端不可用时，系统会在日志中给出 `WARN`，并对可兼容的新索引自动回退到纯 Java ZSTD
- 如果历史索引已经写成依赖 native 后端的 `V3` 格式，需要在支持 native ZSTD 的节点上打开，或先重建/重写为兼容格式

尝试在已有索引上动态更新时，会报类似错误：
```text
final <index-name> setting [index.compression.zstd.jni], not updateable
```


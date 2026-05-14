---
title: "数据汇总"
date: 0001-01-01
summary: "数据汇总 #   先读概述：如果您尚未了解 Rollup 的优势与应用场景，建议先阅读 数据生命周期 中的 Rollup 部分。
 数据汇总或上卷（Rollup），对于时序场景类的数据，往往会有大量的非常详细的聚合指标，随着时间的图推移，存储将持续增长。汇总功能可以将旧的、细粒度的数据汇总为粗粒度格式以进行长期存储。通过将数据汇总到一个单一的文档中，可以大大降低历史数据的存储成本。 Easysearch 的 rollup 具备一些独特的优势，可以自动对 rollup 索引进行滚动而不用依赖其他 API 去单独设置，并且在进行聚合查询时支持直接搜索原始索引，做到了对业务端的搜索代码完全兼容，从而对用户无感知。
支持的聚合类型 #  对数值类型字段支持的聚合
 avg sum max min value_count percentiles  对 keyword 类型字段提供 terms 聚合。
对 date 类型字段 除了 date_histogram 聚合，还支持 date_range 聚合。(v1.10.0)
查询 rollup 数据时，增加支持 Filter aggregation，某些场景可以用来替代 query 过滤数据。(v1.10.1)
增加针对个别字段自定义 special_metrics 指标的配置项。 (v1.10.1)
增加支持 Bucket sort aggregation。 (v1.10.1)
混合查询原始索引和 rollup 索引时，返回的 response 里增加了 origin 参数，表示包含 rollup 数据。(v1."
---


# 数据汇总

> **先读概述**：如果您尚未了解 Rollup 的优势与应用场景，建议先阅读 [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}}) 中的 Rollup 部分。

数据汇总或上卷（Rollup），对于时序场景类的数据，往往会有大量的非常详细的聚合指标，随着时间的图推移，存储将持续增长。汇总功能可以将旧的、细粒度的数据汇总为粗粒度格式以进行长期存储。通过将数据汇总到一个单一的文档中，可以大大降低历史数据的存储成本。<br/>
Easysearch 的 rollup 具备一些独特的优势，可以自动对 rollup 索引进行滚动而不用依赖其他 API 去单独设置，并且在进行聚合查询时支持直接搜索原始索引，做到了对业务端的搜索代码完全兼容，从而对用户无感知。

## 支持的聚合类型
对数值类型字段支持的聚合
* avg
* sum
* max
* min
* value_count
* percentiles

对 keyword 类型字段提供 terms 聚合。

对 date 类型字段 除了 date_histogram 聚合，还支持 date_range 聚合。(v1.10.0)

查询 rollup 数据时，增加支持 Filter aggregation，某些场景可以用来替代 query 过滤数据。(v1.10.1)

增加针对个别字段自定义 special_metrics 指标的配置项。 (v1.10.1)

增加支持 Bucket sort aggregation。 (v1.10.1)

混合查询原始索引和 rollup 索引时，返回的 response 里增加了 `origin` 参数，表示包含 rollup 数据。(v1.10.1)

Rollup 查询 API 提供了 debug 参数，显示 Easysearch 内部执行的查询语句。(v1.10.1)

## 使用汇总（rollup）的先决条件

必须安装索引生命周期管理插件，rollup 属于该插件功能的一部分。

要汇总的指标索引必须具备 date 类型的字段。

## 全局启用或禁用 rollup 搜索的设置

> rollup 功能在 index-management 模块里。
> 
>  从 1.15.2 版本开始，index-management 已经成为 modules 的一部分，不需要单独安装插件。

从 1.10.0 版本开始，索引生命周期管理插件不再默认启用 rollup 搜索功能，如果想启用搜索 rollup 搜索功能，需要设置
```auto
PUT /_cluster/settings
{
    "transient": {
      "rollup.search.enabled": true
    }
}

```

## 通配符方式批量启动停止 rollup job
从 1.10.0 版本开始，创建 rollup job 之后不再自动启动，需要手动启动，但是可以通过通配符的方式批量启动、停止 job

```auto
POST _rollup/jobs/rollup*/_start

POST _rollup/jobs/rollup*/_stop

```

## 设置 rollup 索引自动滚动的条数
达到此条数就会自动滚动新的索引
```auto
PUT /_cluster/settings
{
    "transient": {
      "rollup.max_docs": 10000000
    }
}

```

## 新增 `ROLLUP_SEARCH_MAX_COUNT` 配置

从 1.10.0 版本开始，新增了 `ROLLUP_SEARCH_MAX_COUNT` 配置项，用于控制 Rollup 在运行 Job 时收集历史数据的最大并发分片请求数。这个配置项可以帮助你优化 Rollup 任务的性能，并避免集群资源过载。

## 设置 Rollup 查询的时间范围上限
从 1.10.1 版本开始，可以通过 `rollup.hours_before` 集群配置项，设置 rollup 数据的查询时间范围上限，默认值是 1，表示 针对 rollup 数据的查询不会超过当前时间的 1 小时之前，
此参数的使用场景：
1. 在混合查询时将 rollup 数据和原始数据的查询分割，防止查询的时间范围和数据不匹配。
2. 限制 rollup 数据的查询时间上限。

使用示例：
```auto
PUT /_cluster/settings
{
  "transient": {
    "rollup.hours_before": 24

  }
}
```
设置后，对 rollup 数据的查询范围上限不会超过 24 小时之前。

#### 功能：
- **控制并发请求数**：限制 Rollup 任务在执行搜索请求时的最大并发分片请求数。
- **动态调整**：支持在集群运行时动态调整，无需重启集群。
- **默认值**：`2`，即默认情况下，Rollup 任务最多会同时发送 2 个并发分片请求。

#### 示例：
```auto
PUT /_cluster/settings
{
    "transient": {
      "rollup.search.max_count": 2
    }
}
```

在这个例子中，`ROLLUP_SEARCH_MAX_COUNT` 被设置为 `2`，表示 Rollup 任务在执行搜索请求时，最多会同时发送 2 个并发分片请求。

#### 配置建议：
- **小规模集群**：建议设置为较小的值（如 `2`），以避免资源竞争。
- **大规模集群**：可以适当增加该值（如 `4`），以提高并发性能。
- **动态调整**：根据集群负载情况动态调整该值，以优化性能和资源利用率。


#### rollup 自动滚动后的索引列表
```
health status index                          uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   rollup1_.infini_metrics-000001 aNET5la8QBa8AtCAtplPAw   1   0     179783            0        1gb            1gb
green  open   rollup1_.infini_metrics-000002 7TDSIvWvSvKuKbB0DH-J0w   1   0     179170            0        1gb            1gb
green  open   rollup1_.infini_metrics-000003 MhtBEj_-RgCg29gAbkbg6g   1   0     178297            0        1gb            1gb
green  open   rollup1_.infini_metrics-000004 -Gfp80nUR5Cz-XnFy3F0rw   1   0     178297            0        1gb            1gb
green  open   rollup1_.infini_metrics-000005 CuhhEf9SQ--yzMkG0QgLQw   1   0     177665            0        1gb            1gb
green  open   rollup1_.infini_metrics-000006 9LqFJ28XRKexgwlMgIy3Bg   1   0     178624            0        1gb            1gb
green  open   rollup1_.infini_metrics-000007 XkrPz8DDSRSq8xhnWo3nbA   1   0     180581            0        1gb            1gb
green  open   rollup1_.infini_metrics-000008 xrfETulmT7OZnfcfVrIJTw   1   0     180050            0        1gb            1gb

```

## 汇总（rollup） API：

* 创建或替换索引汇总任务
* 获取索引汇总任务
* 删除索引汇总任务
* 启动或停止索引汇总任务
* 解释索引汇总任务

### 创建或更新索引汇总任务

创建或自动替换一个索引汇总任务

PUT _rollup/jobs/<rollup_id>?replace

<rollup_id>: 为您的 Rollup 任务指定一个唯一标识符。

?replace: 如果携带此参数，将替换已存在的 Rollup 所有配置，请谨慎使用。

#### 请求示例
下面的例子会创建一个自动滚动的 rollup 索引
```auto
PUT _rollup/jobs/rollup_node_stats
{
  "rollup": {
    "source_index": ".infini_metrics",
    "target_index": "rollup_node_stats_{{ctx.source_index}}",
    "timestamp": "timestamp",
    "continuous": true,
    "page_size": 200,
    "cron": "*/5 1-23 * * *",
    "timezone": "UTC+8",
    "stats": [
      {
        "max": {}
      },
      {
        "min": {}
      },
      {
        "value_count": {}
      }
    ],
    "interval": "1m",
    "identity": [
      "metadata.labels.cluster_id",
      "metadata.labels.cluster_uuid",
      "metadata.category",
      "metadata.labels.node_id",
      "metadata.labels.transport_address"
    ],
    "attributes": [
      "agent.*",
      "metadata.*"
    ],
    "filter": {
      "metadata.name": "node_stats"
    },
    "metrics": [
      "payload.elasticsearch.node_stats.*"
    ],
    "special_metrics": [
      {
        "source_field": "payload.elasticsearch.node_stats.jvm.mem.heap_used_in_bytes",
        "metrics": [
          {
            "avg": {}
          },
          {
            "max": {}
          },
          {
            "min": {}
          },
          {
            "percentiles": {}
          }
        ]
      }
    ],
    "write_optimization": true,
    "field_abbr": true
  }
}
```

#### 参数详细说明

| 参数 | 解释                                    | 示例值说明                                                                    |
|------|---------------------------------------|--------------------------------------------------------------------------|
| `source_index` | 源索引，数据将从此索引中收集                        | `.infini_metrics`: 表示从名为 .infini_metrics 的索引中收集数据                        |
| `target_index` | 目标索引，汇总数据将存储在此索引中                     | `rollup1_{{ctx.source_index}}`: 动态生成目标索引名，使用源索引名作为一部分；当前版本已不要求目标索引必须以 `rollup` 开头 |
| `timestamp` | 用于时间聚合的时间戳字段                          | `timestamp`: 指定用于时间聚合的字段名                                                |
| `continuous` | 设置为 true，表示这是一个连续的 rollup 作业，默认为false | `true`: 任务将持续运行，而不是一次性执行                                                 |
| `page_size` | 每次处理的文档数量                             | `1000`: 每批处理1000个文档，可根据系统性能调整                                            |
| `cron` | 作业运行的调度表达式                            | `*/10 1-23 * * *`: 每10分钟执行一次，在1点到23点之间                                   |
| `timezone` | 用于 cron 表达式的时区                        | `UTC+8`: 使用UTC+8时区（如北京时间）                                                |
| `stats` | 要计算的统计信息                              | 示例中计算最大值和计数，可根据需求添加其他统计如avg, min等                                        |
| `interval` | 时间聚合的间隔                               | `1m`: 每分钟聚合一次数据                                                          |
| `identity` | 标识字段，用于分组聚合                           | 列出的字段将用于创建唯一的聚合组                                                         |
| `attributes` | 要包含在 rollup 中的属性字段                    | `agent.*`, `metadata.*`: 包含所有以agent和metadata开头的字段                        |
| `metrics` | 要汇总的指标字段                              | `payload.elasticsearch.index_stats.*`: 汇总所有index_stats下的指标               |
| `special_metrics` | 对特别关注的指标配置更丰富的统计方式                    | 专门对 JVM 堆内存使用量这个字段计算 `avg max min percentiles` 4 个指标，不再计算 `stats` 里的默认指标 |
| `exclude` | 要排除的指标字段                              | `payload.elasticsearch.index_stats.routing.*`: metrics 会排除符合模式的字段        |
| `filter` | 用于过滤源文档的条件                            | `{"metadata.name": "index_stats"}`: 只处理metadata.name为index_stats的文档      |
| `write_optimization` | 启用后采用自动生成文档 ID 的策略，提升写入速度             | `true`: 启用写入优化（1.12.0 版本增加）                                              |
| `field_abbr` | 启用字段名缩写，可降低 rollup 文档、mapping 与批量写入过程中的字段名开销 | `true`: 启用字段名缩写（1.12.0 版本增加）；当前版本主要对使用通配符展开的 `metrics` 字段生效 |
| `is_continue` | 当创建的 Rollup 已经有历史数据并要断点续跑时配置          | `true`: 启用断点续跑，默认不配置为 `false`（1.12.3 版本增加）                               |

#### 注意事项

1. `cron` 表达式需要谨慎设置，确保不会对系统造成过大负担。
2. `page_size` 应根据您的系统性能和数据量来调整。
3. `identity` 字段的选择会影响聚合的粒度，请根据您的分析需求仔细选择。
4. 使用通配符（如 `agent.*`）时要注意，这可能会包含大量字段，影响性能。
5. `filter` 可以有效减少处理的数据量，提高效率。
6. `write_optimization` 和 `field_abbr` 是从 1.12.0 新增的参数。
7. `target_index` 只是目标索引名称模板，当前版本不会再通过固定的 `rollup` 名称前缀识别 rollup 索引；rollup 搜索链路主要基于索引元数据中的 `index.rollup_index` 设置判断目标索引。
8. `field_abbr` 当前最适合与通配符形式的 `metrics` 一起使用；如果使用显式 `metrics` 列表，字段名缩写带来的收益会明显减弱。

#### `field_abbr` 的主要收益与适用场景

`field_abbr` 的作用不是改变聚合结果，而是把 rollup 目标文档里原本很长的指标字段名缩写成较短的字段名，从而减少字段名重复带来的开销。对于包含大量嵌套 JSON 路径、并且 `metrics` 通过通配符批量展开的场景，这个优化最明显。

主要收益包括：

1. 缩小 rollup 文档 `_source` 体积。字段路径越长、每条文档里重复出现得越多，收益越明显。
2. 缩小 rollup 索引 mapping 体积。大量动态展开的 metrics 会生成很多字段，缩写后 mapping 元数据会更紧凑。
3. 降低 bulk 写入时的请求体、序列化和临时内存开销。对于持续写入、metrics 数量很多的任务，这通常比“单条文档省了几个字节”更有实际意义。
4. 在 wildcard metrics 场景下，通常还能带来一定的 rollup 索引存储节省，但这个收益一般小于 `_source` 和 mapping 的节省幅度。

当前实现有两个边界需要注意：

1. `field_abbr` 目前主要对通配符展开出来的 `metrics` 生效；如果使用显式 `metrics` 列表，当前实现仍会保留原始字段名，收益会明显减弱。
2. 不应把 `field_abbr` 理解成“聚合计算时 docvalues 内存显著下降”的优化。它主要优化的是字段名本身带来的文档体积、mapping 体积和写入链路开销，而不是数值聚合本身的计算模型。

### 获取索引汇总任务

请求
```auto
GET _rollup/jobs/<rollup_id>
```

响应
```auto
{
  "_id": "rollup4",
  "_version": 3,
  "_seq_no": 5,
  "_primary_term": 1,
  "rollup": {
    "rollup_id": "rollup4",
    "enabled": false,
    "schedule": {
      "interval": {
        "start_time": 1718105163766,
        "period": 1,
        "unit": "Minutes",
        "schedule_delay": 0
      }
    },
    "last_updated_time": 1718105163767,
    "enabled_time": null,
    "description": "Example rollup job",
    "schema_version": 17,
    "source_index": "test-data",
    "target_index": "rollup4-test",
    "metadata_id": "nuMNB5ABnvQbLaVPM5qO",
    "page_size": 200,
    "delay": 0,
    "continuous": false,
    "dimensions": [
      {
        "date_histogram": {
          "fixed_interval": "1m",
          "source_field": "@timestamp",
          "target_field": "@timestamp",
          "timezone": "UTC"
        }
      }
    ],
    "metrics": [
      {
        "source_field": "passenger_count",
        "metrics": [
          {
            "avg": {}
          },
          {
            "sum": {}
          },
          {
            "max": {}
          },
          {
            "min": {}
          },
          {
            "value_count": {}
          }
        ]
      }
    ]
  }
}
```

### 删除索引汇总任务

请求
```auto
DELETE _rollup/jobs/<rollup_id>
```
响应
```auto
{
  "_index": ".easysearch-ilm-config",
  "_type": "_doc",
  "_id": "rollup4",
  "_version": 7,
  "result": "deleted",
  "forced_refresh": true,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 23,
  "_primary_term": 1
}
```

### 启动或停止索引汇总任务

请求
```auto
POST _rollup/jobs/<rollup_id>/_start
POST _rollup/jobs/<rollup_id>/_stop
```
响应
```auto
{
  "acknowledged": true
}
```

### 解释索引汇总任务

请求
```auto
GET _rollup/jobs/<rollup_id>/_explain
```
响应
```auto
{
  "rollup1": {
    "metadata_id": "VCAnj5IBQpC3NDmZwgL7",
    "rollup_metadata": {
      "rollup_id": "rollup1",
      "last_updated_time": 1729151524871,
      "continuous": {
        "next_window_start_time": 1728196860000,
        "next_window_end_time": 1728196920000
      },
      "status": "started",
      "failure_reason": null,
      "stats": {
        "pages_processed": 77014,
        "documents_processed": 135026197,
        "rollups_indexed": 26812185,
        "index_time_in_millis": 38094616,
        "search_time_in_millis": 88930100
      }
    }
  }
}
```

| 字段名 | 类型 | 描述                              |
|--------|------|---------------------------------|
| `metadata_id` | string | rollup 元数据的唯一标识符                |
| `rollup_metadata.rollup_id` | string | rollup 作业的 ID                   |
| `rollup_metadata.last_updated_time` | long | 上次更新时间的 Unix 时间戳（毫秒）            |
| `rollup_metadata.continuous.next_window_start_time` | long | 下一个 rollup 窗口的开始时间（Unix 时间戳，毫秒） |
| `rollup_metadata.continuous.next_window_end_time` | long | 下一个 rollup 窗口的结束时间（Unix 时间戳，毫秒） |
| `rollup_metadata.status` | string | rollup 作业当前的状态                  |
| `rollup_metadata.failure_reason` | string/null | 如果作业失败，显示失败原因；否则为 null |
| `rollup_metadata.stats.pages_processed` | long | 已处理的页面数                         |
| `rollup_metadata.stats.documents_processed` | long | 已处理的文档数                         |
| `rollup_metadata.stats.rollups_indexed` | long | 已索引的 rollup 文档数                 |
| `rollup_metadata.stats.index_time_in_millis` | long | 索引所用的总时间（毫秒）                    |
| `rollup_metadata.stats.search_time_in_millis` | long | 搜索所用的总时间（毫秒）                    |

### 修改已经运行 Job 的 interval 和 page_size 参数 (1.13.0)

在线上环境运行 Rollup 时会遇到这种情况：由于机器硬件资源或其他原因发现某个 Rollup 压力较大，需要调整最小时间区间或单次请求条数。

从 1.13.0 版本开始，可以使用 POST _rollup/jobs/<rollup_id> API 进行更新，此操作内部会自动执行 
1. 停止 Job
2. 更新 参数
3. 重启 Job
4. 运行 Job 时 索引数据会增加 unique 字段标识当前 Job 批次的数据，方便数据排重

补充说明：

- 当前版本允许在保留已有 `metrics` 定义不变的前提下，向 Rollup job 追加新的指标字段。
- 不允许删除已有指标字段，也不允许修改已有指标字段的聚合定义；如需变更已有指标语义，请创建新的 Rollup job。

请求
```auto
POST  _rollup/jobs/rollup_node_stats  
{
  "page_size": 100,
  "interval": "5m"
}
```
响应
```auto
{
  "acknowledged": true
}
```

## 如何使用 rollup 索引进行搜索

无需特意搜索 rollup 索引，只需使用标准的 search API 对原始目标索引进行搜索。查询时，请确保查询符合目标索引的约束条件。
例如，如果你没有在某个字段上设置 terms 聚合，那么你不会收到该字段的 terms 聚合结果。如果你没有设置 maximum 聚合，也不会收到 maximum 聚合的结果。

无法访问目标索引中数据的内部结构，因为插件会在后台自动重写查询以适应目标索引。这是为了确保对源索引和目标索引使用相同的查询。

请求示例
```auto
GET target-test/_search
{
  "size": 0,
  "aggs": {
    "a": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "1h"
      }
    },
    "total_passenger_count": {
      "sum": {
        "field": "passenger_count"
      }
    }
  }
}
```

响应
```auto
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "a": {
      "buckets": [
        {
          "key_as_string": "2023-08-01T10:00:00.000Z",
          "key": 1690884000000,
          "doc_count": 10
        }
      ]
    },
    "total_passenger_count": {
      "value": 38
    }
  }
}
```

### 注意

不能在同一个搜索请求中同时搜索汇总索引和非汇总索引。

## 为自定义用户赋予rollup 权限

以下示例表示创建了一个自定义用户，这个用户只对 test 开头的索引具备所有权限，并赋予他 rollup 的权限，授权后，test_user将可以创建 rollup job，并对rollup 开头的索引有查询权限。

```auto
PUT /_security/role/test_role
{
  "indices": [
    {
      "names": [ "test*"],
      "privileges": [ "*" ]
    }
  ]
}

PUT /_security/user/test_user
{
  "password" : "123456",
  "roles" : [ "test_role","rollup_all" ]
}
```

## 断点续跑 (1.13.0)

当连续 Rollup Job 已经运行过，并且已经生成历史 metadata 时，可以通过 `is_continue: true` 重建同名 Job，让新 Job 从旧的处理中断点继续执行，避免从头重复计算。

### 使用方式

断点续跑的典型步骤如下：

1. 先让旧 Rollup Job 至少成功运行过一次，确保 `.ilm-config` 中已经存在对应的 `rollup_metadata`。
2. 停止旧 Job：

```json
POST /_rollup/jobs/my_rollup_job/_stop
```

3. 删除旧 Job：

```json
DELETE /_rollup/jobs/my_rollup_job
```

4. 使用相同的 `rollup_id` 重新创建 Job，并在配置中增加 `is_continue: true`：

```json
PUT /_rollup/jobs/my_rollup_job?refresh=true
{
  "rollup": {
    "source_index": "my_source_index",
    "target_index": "my_rollup_target",
    "timestamp": "ts",
    "page_size": 100,
    "delay": 0,
    "cron": "* * * * *",
    "timezone": "UTC",
    "continuous": true,
    "interval": "1m",
    "identity": [
      "region"
    ],
    "attributes": [],
    "filter": {},
    "metrics": [
      "value"
    ],
    "stats": [
      { "sum": {} },
      { "max": {} },
      { "min": {} },
      { "value_count": {} },
      { "avg": {} }
    ],
    "write_optimization": true,
    "field_abbr": false,
    "is_continue": true
  }
}
```

5. 启动新 Job：

```json
POST /_rollup/jobs/my_rollup_job/_start
```

6. 通过 explain 查看是否已经挂回旧 metadata，并确认当前断点：

```json
GET /_rollup/jobs/my_rollup_job/_explain
```

典型返回中可重点关注：

- `metadata_id`
- `rollup_metadata.continuous.next_window_start_time`
- `rollup_metadata.continuous.next_window_end_time`

### 使用前提

当前版本的断点续跑依赖以下条件：

1. 新旧 Job 的 `rollup_id` 必须相同。
2. 历史 `rollup_metadata` 必须仍然存在于 `.ilm-config` 中。
3. 历史 target 索引通常也应保留，避免重建后与历史结果脱节。

### 关于 `window_start_time`

重建成功后，系统会自动把历史 metadata 中的最新断点时间回填到 Job 文档的 `window_start_time` 字段中。

这个字段主要用于观察当前 Job 的断点位置，例如：

- 当前 Job 是从哪个时间窗口开始继续处理的
- 重建后是否已经成功挂回历史 metadata

可以通过查看 Job 文档中的 `window_start_time`，或 `/_rollup/jobs/{id}/_explain` 返回中的 `rollup_metadata.continuous.next_window_start_time` 来确认当前恢复点。

### 注意事项

1. `is_continue` 不是“从任意指定时间开始”的通用恢复参数，而是“重建同名 Job 时复用历史 metadata”的恢复开关。
2. 当前版本不会通过 REST 请求直接接收用户手工传入的 `window_start_time`。
3. 如果历史 metadata 已被删除，或无法重新关联到新 Job，则无法继续从旧断点恢复。


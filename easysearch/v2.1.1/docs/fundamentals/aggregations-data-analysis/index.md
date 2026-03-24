---
title: "聚合与数据分析"
date: 0001-01-01
description: "聚合的基本概念、桶聚合、指标聚合、管道聚合与数据分析。"
summary: "聚合与数据分析 #  聚合（Aggregations）是 Easysearch 用于做统计与分析的核心能力，可以在一次请求中同时完成&quot;搜索 + 统计&quot;。本页介绍聚合的概念、类型与用法。
聚合可以解决什么问题？ #  典型场景：
 统计：总数、平均值、最大/最小值、百分位数 分布：按字段分桶（如按状态、地区、时间段统计数量） 下钻分析：先按大维度分组，再在组内做更细粒度分析  和传统数据库的类比：
 可以粗略类比为 GROUP BY + 聚合函数，但聚合可以与全文搜索、过滤等能力组合，且天然适应分布式环境。  聚合请求的基本结构 #  一个最小的&quot;搜索 + 聚合&quot;请求大致长这样：
{ &#34;query&#34;: { &#34;term&#34;: { &#34;status&#34;: &#34;success&#34; } }, &#34;aggs&#34;: { &#34;by_host&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;host&#34; } } } } 结构说明：
 query：决定哪些文档参与统计（可为空，表示全量） aggs：定义一个或多个聚合，每个聚合都有一个名字（如 by_host）  聚合的结果会出现在响应体的 aggregations 区域，与 hits（命中的文档）并列。
桶（Bucket）与指标（Metric） #  大多数聚合可以拆成两类：
 桶聚合（Bucket Aggregations）：把文档分到不同&quot;桶&quot;里
例如：按字段值分桶（terms）、按数值区间分桶（range）、按时间间隔分桶（date_histogram）等。 指标聚合（Metric Aggregations）：对某个字段在桶内做统计"
---


# 聚合与数据分析

聚合（Aggregations）是 Easysearch 用于做统计与分析的核心能力，可以在一次请求中同时完成"搜索 + 统计"。本页介绍聚合的概念、类型与用法。

## 聚合可以解决什么问题？

典型场景：

- 统计：总数、平均值、最大/最小值、百分位数
- 分布：按字段分桶（如按状态、地区、时间段统计数量）
- 下钻分析：先按大维度分组，再在组内做更细粒度分析

和传统数据库的类比：

- 可以粗略类比为 `GROUP BY + 聚合函数`，但聚合可以与全文搜索、过滤等能力组合，且天然适应分布式环境。

## 聚合请求的基本结构

一个最小的"搜索 + 聚合"请求大致长这样：

```json
{
  "query": {
    "term": { "status": "success" }
  },
  "aggs": {
    "by_host": {
      "terms": {
        "field": "host"
      }
    }
  }
}
```

结构说明：

- `query`：决定哪些文档参与统计（可为空，表示全量）
- `aggs`：定义一个或多个聚合，每个聚合都有一个名字（如 `by_host`）

聚合的结果会出现在响应体的 `aggregations` 区域，与 `hits`（命中的文档）并列。

## 桶（Bucket）与指标（Metric）

大多数聚合可以拆成两类：

- **桶聚合（Bucket Aggregations）**：把文档分到不同"桶"里  
  例如：按字段值分桶（terms）、按数值区间分桶（range）、按时间间隔分桶（date_histogram）等。
- **指标聚合（Metric Aggregations）**：对某个字段在桶内做统计  
  例如：`avg`、`sum`、`min`、`max`、`percentiles` 等。

通常会把两者组合使用：

```json
{
  "aggs": {
    "by_status": {
      "terms": { "field": "status" },
      "aggs": {
        "avg_latency": {
          "avg": { "field": "latency" }
        }
      }
    }
  }
}
```

含义：

- 先按 `status` 分桶（每个状态一个桶）
- 在每个桶内，再计算 `latency` 的平均值

## 常见模式：只要聚合，不要文档

有时候你只需要统计结果，不关心具体命中的文档，可以把 `size` 设为 0：

```json
{
  "size": 0,
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-1d/d"
      }
    }
  },
  "aggs": {
    "by_level": {
      "terms": { "field": "level" }
    }
  }
}
```

这样可以减少响应体体积和网络传输开销。

## 与查询结合的典型用法

聚合总是基于"参与统计的文档集合"来计算，因此 `query` 非常关键：

- 通过 `query` / `filter` 限定人群/时间范围/业务条件
- 通过 `aggs` 在这一子集上做统计和分布分析

例如：统计最近 7 天内，每个应用的错误日志数量：

```json
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        { "term": { "log_level": "ERROR" } },
        { "range": { "timestamp": { "gte": "now-7d/d" } } }
      ]
    }
  },
  "aggs": {
    "by_app": {
      "terms": { "field": "app_name" }
    }
  }
}
```

---

## 桶聚合（Bucket Aggregations）

桶聚合（Bucket Aggregations）负责"分组"：把满足查询条件的文档划分到不同的桶中，后续可以在桶内再做指标聚合或进一步下钻。

### 什么时候用桶聚合？

典型问题：

- 按状态/地区/应用等字段查看分布情况
- 按时间窗口（天/小时）查看趋势
- 按数值区间（价格段、年龄段等）汇总

桶本身不做数值计算，但提供"分组维度"；通常会和指标聚合（Metrics）组合使用。

### terms：按字段值分桶

`terms` 是最常用的桶聚合，用于按字段的不同取值聚合，例如按状态统计数量：

```json
{
  "size": 0,
  "aggs": {
    "by_status": {
      "terms": {
        "field": "status",
        "size": 10
      }
    }
  }
}
```

要点：

- 适合 keyword/数值等可精确匹配的字段
- `size` 控制返回的桶个数（按文档数从高到低排序）

常见注意点：

- 高频值很多时（如用户 ID），`terms` 可能产生非常多桶 → 建议只对维度有限的字段使用（状态、地区、应用名等）
- 如需对极高基数字段做分析，可考虑预聚合、采样或专门的数据建模

### histogram：数值直方图

`histogram` 用于把数值字段按固定间隔分桶，例如按 10 元价格区间统计订单数：

```json
{
  "size": 0,
  "aggs": {
    "price_histogram": {
      "histogram": {
        "field": "price",
        "interval": 10
      }
    }
  }
}
```

要点：

- 适合数值字段
- `interval` 控制桶宽度，选择合适的间隔可以平衡"细腻度"和性能

### date_histogram：时间序列分桶

`date_histogram` 是时间序列分析的核心，用于按固定时间间隔（天/小时/分钟等）划分桶：

```json
{
  "size": 0,
  "aggs": {
    "per_day": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      }
    }
  }
}
```

常用参数：

- `calendar_interval`：按自然时间单位（year/month/week/day/hour 等）
- `fixed_interval`：按固定毫秒数间隔（适合更精细控制）

典型用法：

- 结合错误码或业务维度做趋势图
- 作为折线图/柱状图的 X 轴

### range：自定义范围分桶

`range` 允许你手动定义数值或日期区间：

```json
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 500 },
          { "from": 500 }
        ]
      }
    }
  }
}
```

适合：

- 需要特定业务切分方式（如"低价/中价/高价"）
- 不希望桶边界由系统自动划分

### 桶内再嵌套桶与指标

桶聚合可以嵌套使用，实现多维下钻，例如：

```json
{
  "size": 0,
  "aggs": {
    "by_app": {
      "terms": { "field": "app_name" },
      "aggs": {
        "per_day": {
          "date_histogram": {
            "field": "timestamp",
            "calendar_interval": "day"
          }
        }
      }
    }
  }
}
```

结构含义：

- 第一层按应用分桶（by_app）
- 第二层在每个应用桶内，再按日期分桶（per_day）

在任何桶内，你都可以继续添加指标聚合（如平均值、总和等），实现"多维分组 + 多个指标"的报表。

### 选型与实践建议

- 优先用 `terms` 处理离散维度（状态、地区、应用名等）
- 对数值/时间做趋势图与直方图时，使用 `histogram`/`date_histogram`
- 需要特定业务区间时使用 `range`，并确保区间覆盖与顺序正确
- 控制桶的数量与嵌套深度，避免一次性生成过多桶带来的内存与网络开销

---

## 指标聚合（Metric Aggregations）

指标聚合用于在桶内（或整个结果集上）计算各种统计值，比如平均值、总和、最值、计数、百分位等。

### count：文档数量

最简单的"指标"其实是每个桶里的文档数量，这在绝大多数桶聚合结果里会自动给出（`doc_count` 字段）。

例如，在 `terms` 桶聚合结果中，每个桶都会包含：

- `key`：桶的键（比如状态值）
- `doc_count`：落入该桶的文档数量

通常不需要额外配置。

### avg / sum / min / max：基础统计

这些是最常用的数值统计指标，示例：按应用统计最近一天平均延迟和总请求数：

```json
{
  "size": 0,
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-1d/d"
      }
    }
  },
  "aggs": {
    "by_app": {
      "terms": { "field": "app_name" },
      "aggs": {
        "avg_latency": {
          "avg": { "field": "latency" }
        },
        "total_requests": {
          "sum": { "field": "requests" }
        },
        "min_latency": {
          "min": { "field": "latency" }
        },
        "max_latency": {
          "max": { "field": "latency" }
        }
      }
    }
  }
}
```

适用场景：

- 指标监控、报表、容量评估
- 在业务维度（应用、地区、用户群等）下做统计

### percentiles / percentile_ranks：百分位

百分位在性能分析中非常重要（P50/P95/P99 等），可用 `percentiles` 指标聚合实现：

```json
{
  "size": 0,
  "aggs": {
    "latency_percentiles": {
      "percentiles": {
        "field": "latency",
        "percents": [50, 95, 99]
      }
    }
  }
}
```

含义：

- P50：50% 的请求延迟小于等于该值
- P95：95% 的请求延迟小于等于该值

`percentile_ranks` 则反过来：给定一些值，计算它们分别处于全体样本的哪一百分位：

```json
{
  "size": 0,
  "aggs": {
    "latency_ranks": {
      "percentile_ranks": {
        "field": "latency",
        "values": [100, 300, 500]
      }
    }
  }
}
```

### 组合多个指标

你可以在同一个桶下组合多个指标，例如对每个状态统计：

- 文档数（`doc_count`）
- 平均延迟（`avg`）
- 错误率（通过额外的 filter/脚本或预先建模实现）

结构类似：

```json
{
  "aggs": {
    "by_status": {
      "terms": { "field": "status" },
      "aggs": {
        "avg_latency": {
          "avg": { "field": "latency" }
        },
        "latency_percentiles": {
          "percentiles": {
            "field": "latency",
            "percents": [50, 95, 99]
          }
        }
      }
    }
  }
}
```

### 实践建议

- 指标字段类型应为数值或日期类型，避免在 text/keyword 上做无意义的指标计算
- 对高基数/高频更新的指标字段，要注意聚合的成本（可通过采样、预聚合或索引设计优化）
- 在监控和报表场景下，先从少量关键指标开始，逐步增加复杂度，避免一开始就请求过多指标导致性能问题

---

## 管道聚合（Pipeline Aggregations）

管道聚合（Pipeline Aggregations）是一类特殊的聚合，它们不直接处理文档，而是在其他聚合的结果之上进行计算。管道聚合可以用于计算衍生指标、移动平均值、百分比变化等。

### 管道聚合的类型

管道聚合主要分为两类：

1. **父级管道聚合**：在父聚合的结果之上进行计算，例如计算每个桶的衍生值
2. **兄弟管道聚合**：在兄弟聚合的结果之上进行计算，例如计算两个聚合之间的差值

常见的管道聚合包括：

- `derivative`：计算父聚合的衍生值（一阶导数）
- `moving_avg`：计算移动平均值
- `bucket_script`：使用脚本对多个聚合结果进行计算
- `bucket_selector`：根据条件选择桶
- `cumulative_sum`：计算累积和
- `percentile_bucket`：计算百分位桶

### 衍生值（Derivative）

`derivative` 聚合计算父聚合的衍生值。这对于时间序列数据特别有用，可以计算变化率。

例如，计算每月销售额的月度变化：

```json
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_deriv": {
          "derivative": {
            "buckets_path": "sales"
          }
        }
      }
    }
  }
}
```

`derivative` 会计算相邻桶之间的差值。第一个桶没有前一个桶，所以不会有衍生值。

### 移动平均值（Moving Average）

`moving_avg` 聚合计算移动平均值，这对于平滑时间序列数据非常有用。

例如，计算销售额的 7 天移动平均值：

```json
{
  "aggs": {
    "sales_per_day": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_moving_avg": {
          "moving_avg": {
            "buckets_path": "sales",
            "window": 7
          }
        }
      }
    }
  }
}
```

`window` 参数指定移动窗口的大小。移动平均值会平滑数据，减少短期波动的影响。

### 桶脚本（Bucket Script）

`bucket_script` 聚合允许你使用脚本对多个聚合结果进行计算。

例如，计算每个月的销售额增长率：

```json
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_growth": {
          "bucket_script": {
            "buckets_path": {
              "prevMonthSales": "sales[_key - 1m]",
              "currentSales": "sales"
            },
            "script": "params.currentSales - params.prevMonthSales"
          }
        }
      }
    }
  }
}
```

### 桶选择器（Bucket Selector）

`bucket_selector` 聚合可以根据条件过滤或移除某些桶。

例如，只保留销售额大于 1000 的月份：

```json
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_bucket_filter": {
          "bucket_selector": {
            "buckets_path": {
              "sales": "sales"
            },
            "script": "params.sales > 1000"
          }
        }
      }
    }
  }
}
```

### 累积求和（Cumulative Sum）

`cumulative_sum` 聚合计算累积和，适合展示累积增长趋势。

例如，计算每月销售额的累计总和：

```json
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "cumulative_sales": {
          "cumulative_sum": {
            "buckets_path": "sales"
          }
        }
      }
    }
  }
}
```

### 同级管道聚合（Sibling Aggregations）

同级管道聚合（如 `avg_bucket`、`sum_bucket` 等）可以在多个桶之间做计算。

例如，计算所有销售月份的平均销售额：

```json
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "total_avg_sales": {
      "avg_bucket": {
        "buckets_path": "sales_per_month>sales"
      }
    }
  }
}
```

### 应用场景

管道聚合特别适用于：

1. **时间序列分析**：计算变化率、移动平均值、趋势等
2. **业务指标计算**：增长率、占比、累计值等
3. **数据过滤**：根据计算结果过滤桶
4. **复杂计算**：需要多个聚合结果参与的计算

### 注意事项

1. **性能影响**：管道聚合会增加查询的计算开销，特别是在大数据集上
2. **内存使用**：某些管道聚合（如移动平均值）需要缓存历史数据
3. **空值处理**：某些桶可能没有值，需要处理空值情况
4. **脚本性能**：使用脚本的管道聚合（如 `bucket_script`）性能可能不如内置聚合

---

## 小结

- 聚合由桶（Buckets）和指标（Metrics）组成
- 桶用于分组，指标用于统计计算
- 聚合可以嵌套使用，实现多维下钻
- 聚合总是基于查询结果进行计算
- 设置 `size: 0` 可以只返回聚合结果，不返回文档
- 管道聚合可以在已有聚合结果之上做二次计算

---

## 参考手册（API 与参数）

详细的聚合类型与参数说明，请查看功能参考中的 API 文档：

- [桶聚合 API 参考]({{< relref "/docs/features/aggregations/bucket-aggregations/" >}})
- [指标聚合 API 参考]({{< relref "/docs/features/aggregations/metric-aggregations/" >}})
- [管道聚合 API 参考]({{< relref "/docs/features/aggregations/pipeline-aggregations/" >}})

### 最佳实践

- [查询调优与慢查询排查]({{< relref "/docs/best-practices/query-tuning.md" >}})：聚合性能优化、分片聚合策略
- [数据建模]({{< relref "/docs/best-practices/data-modeling/" >}})：为聚合而设计的字段结构、反范式权衡


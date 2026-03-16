---
title: "管道聚合"
date: 0001-01-01
summary: "📖 概念与教程请阅读 管道聚合教程
 管道聚合 - API 参考 #  本节详细列出所有管道聚合的参数与用法。
相关资源 #    聚合基础教程  管道聚合教程（概念、最佳实践、常见用法）  聚合场景实践   管道聚合类型 #  管道聚合分为父级聚合（在每个桶内计算）和同级聚合（跨所有桶计算）两类。
父级管道聚合 (8 种) #  父级聚合单独处理每个桶，并将结果写回到每个桶中：
   聚合 说明 常见用途      derivative 计算一阶/二阶导数（变化率） 趋势分析、增长率计算    cumulative_sum 计算累积和 累积总额、运行总计    moving_avg 计算移动平均值 趋势平滑、噪声消除    moving_function 在滑动窗口上执行自定义脚本 自定义移动计算    serial_diff 计算当前桶与前一个桶的差值 同比对比、季节性分析    bucket_script 使用脚本对多个指标进行计算 自定义指标、复杂计算    bucket_selector 根据条件过滤桶 结果过滤、异常检测    bucket_sort 对桶进行排序或截断 排序、分页、TopK 结果    同级管道聚合 (7 种) #  同级聚合跨所有桶计算，生成单个输出值："
---


> 📖 **概念与教程请阅读** [管道聚合教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})

# 管道聚合 - API 参考

本节详细列出所有管道聚合的参数与用法。

## 相关资源

- [聚合基础教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [管道聚合教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})（概念、最佳实践、常见用法）
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

---

## 管道聚合类型

管道聚合分为**父级聚合**（在每个桶内计算）和**同级聚合**（跨所有桶计算）两类。

### 父级管道聚合 (8 种)

父级聚合单独处理每个桶，并将结果写回到每个桶中：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [derivative]({{< relref "derivative.md" >}}) | 计算一阶/二阶导数（变化率） | 趋势分析、增长率计算 |
| [cumulative_sum]({{< relref "cumulative-sum.md" >}}) | 计算累积和 | 累积总额、运行总计 |
| [moving_avg]({{< relref "moving-avg.md" >}}) | 计算移动平均值 | 趋势平滑、噪声消除 |
| [moving_function]({{< relref "moving-function.md" >}}) | 在滑动窗口上执行自定义脚本 | 自定义移动计算 |
| [serial_diff]({{< relref "serial-diff.md" >}}) | 计算当前桶与前一个桶的差值 | 同比对比、季节性分析 |
| [bucket_script]({{< relref "bucket-script.md" >}}) | 使用脚本对多个指标进行计算 | 自定义指标、复杂计算 |
| [bucket_selector]({{< relref "bucket-selector.md" >}}) | 根据条件过滤桶 | 结果过滤、异常检测 |
| [bucket_sort]({{< relref "bucket-sort.md" >}}) | 对桶进行排序或截断 | 排序、分页、TopK 结果 |

### 同级管道聚合 (7 种)

同级聚合跨所有桶计算，生成单个输出值：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [avg_bucket]({{< relref "avg-bucket.md" >}}) | 计算所有桶的指标平均值 | 平均桶指标、汇总分析 |
| [sum_bucket]({{< relref "sum-bucket.md" >}}) | 计算所有桶的指标总和 | 总计、聚合求和 |
| [min_bucket]({{< relref "min-bucket.md" >}}) | 找出所有桶中的最小值 | 最小值检测 |
| [max_bucket]({{< relref "max-bucket.md" >}}) | 找出所有桶中的最大值 | 最大值检测、峰值分析 |
| [stats_bucket]({{< relref "stats-bucket.md" >}}) | 返回所有桶的统计信息 | 统计分析 |
| [extended_stats_bucket]({{< relref "extended-stats.md" >}}) | 返回扩展统计信息（含标准差等） | 扩展统计、方差分析 |
| [percentiles_bucket]({{< relref "percentiles-bucket.md" >}}) | 计算所有桶的百分位数 | 百分位分析、分布统计 |

---

## buckets_path 语法

管道聚合使用 `buckets_path` 参数引用其他聚合的输出：

```
buckets_path = <agg_name>[ > <agg_name> ... ][ .<metric_name> ]
```

| 元素 | 说明 |
|------|------|
| `<agg_name>` | 要引用的聚合名称 |
| `>` | 子选择器，导航到嵌套聚合 |
| `.<metric_name>` | 从多值聚合中选择特定指标 |

### 示例

```json
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_sales": {
          "sum": { "field": "price" }
        },
        "sales_deriv": {
          "derivative": {
            "buckets_path": "total_sales"
          }
        }
      }
    }
  }
}
```

---

## 注意事项

1. 管道聚合**在所有其他同级聚合之后执行**
2. 管道聚合**不能包含子聚合**，但可以链接其他管道聚合
3. 建议将 `min_doc_count` 设置为 0，避免空桶被跳过导致计算错误

---

## buckets_path 进阶

要引用嵌在 `parent_agg` 中的 `child_agg` 的平均价格，请使用 `parent_agg>child_agg.avg`。

参考样例：

- `my_sum.sum`：引用 `my_sum` 聚合中的总和指标
- `popular_tags>my_sum.sum`：引用嵌在 `popular_tags` 聚合下的 `my_sum` 聚合中的 `sum` 指标

> 对于多值指标聚合（如 `stats` 或 `percentiles`），必须在路径中包含指标名称（例如 `.min`）。对于单值指标（如 `sum` 或 `avg`），如果无歧义可以省略指标名称。

### 分组路径示例

以下示例基于日志样本数据。它在 `bytes` 字段的值上创建直方图，在每个直方图分组中求和 `phpmemory` 字段值，并最终使用 `sum_bucket` 管道聚合求和这些分组。 `buckets_path` 遵循 `number_of_bytes>sum_total_memory` 路径从 `number_of_bytes` 父聚合到 `sum_total_memory` 子聚合：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes": {
      "histogram": {
        "field": "bytes",
        "interval": 10000
      },
      "aggs": {
        "sum_total_memory": {
          "sum": {
            "field": "phpmemory"
          }
        }
      }
    },
    "sum_copies": {
      "sum_bucket": {
        "buckets_path": "number_of_bytes>sum_total_memory"
      }
    }
  }
}

```

请注意， `buckets_path` 包含组件聚合的名称。路径是定向的，意味着它们单向传递，从父聚合到子聚合。

管道聚合返回所有分组汇总的总内存：

```
{
  ...
  "aggregations": {
    "number_of_bytes": {
      "buckets": [
        {
          "key": 0,
          "doc_count": 13372,
          "sum_total_memory": {
            "value": 91266400
          }
        },
        {
          "key": 10000,
          "doc_count": 702,
          "sum_total_memory": {
            "value": 0
          }
        }
      ]
    },
    "sum_copies": {
      "value": 91266400
    }
  }
}
```

### 统计路径

你可以将 `buckets_path` 指向一个计数而非值作为其输入。为此，请使用 `_count` 分组路径变量。

以下示例计算来自日志样本数据的字节数的直方图的基本统计信息。它在 `bytes` 字段的值上创建一个直方图，然后在直方图分组中的计数上计算统计信息。

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes": {
      "histogram": {
        "field": "bytes",
        "interval": 10000
      }
    },
    "count_stats": {
      "stats_bucket": {
        "buckets_path": "number_of_bytes>_count"
      }
    }
  }
}
```

结果显示每个分组的文档计数统计信息：

```
{
...
  "aggregations": {
    "number_of_bytes": {
      "buckets": [
        {
          "key": 0,
          "doc_count": 13372
        },
        {
          "key": 10000,
          "doc_count": 702
        }
      ]
    },
    "count_stats": {
      "count": 2,
      "min": 702,
      "max": 13372,
      "avg": 7037,
      "sum": 14074
    }
  }
}
```

## 数据缺口

实际使用中，可以使用嵌套聚合对多个原因进行聚合，原因包括：

- 数值中的缺失值。
- 聚合链中的空分组。
- 计算分组数所需的缺失数值（例如，，某些分组号需要一个或多个前置值才能计算）。

您可以使用 `gap_policy` 属性指定处理缺失数据的策略：跳过缺失数据或将缺失数据替换为零。

`gap_policy` 参数适用于所有管道聚合。

| 参数         | 必选/可选 | 数据类型 | 描述                                                                                   |
| ------------ | --------- | -------- | -------------------------------------------------------------------------------------- |
| `gap_policy` | 可选      | string   | 缺失数据的应用策略。有效值为 skip 和 insert_zeros 。默认值为 skip 。                   |
| `format`     | 可选      | string   | 一个 DecimalFormat 格式化字符串。返回格式化后的输出到聚合的 `value_as_string` 属性中。 |

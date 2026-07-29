---
title: "扩展统计桶聚合（Extended Stats Bucket）"
date: 0001-01-01
summary: "扩展统计桶聚合 #  extended_stats_bucket 聚合是 stats_bucket 同级聚合的更全面的版本。除了 stats_bucket 提供的基本统计度量外，extended_stats_bucket 还计算以下指标：
 平方和 方差 总体方差 抽样方差 标准差 总体标准差 抽样标准差 标准差界限： ** 上限 ** 下限 ** 种群上限 ** 种群下限 ** 采样上限 ** 采样下限  标准差和方差是总体统计量；它们分别始终等于总体标准差和方差。
std_deviation_bounds 对象定义了一个范围，该范围在均值（默认为两个标准差）的上方和下方跨越指定的标准差数量。此对象始终包含在输出中，但仅对正态分布的数据才有意义。在解释这些值之前，请验证您的数据集是否遵循正态分布。
指定的指标必须是数值型，并且同级聚合必须是多分组聚合。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  extended_stats_bucket 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合分组的路径。参见分组路径。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认为 skip 。参见数据间隙。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as_string 属性中的格式化输出。   sigma 可选 Double 非负） 用于计算 std_deviation_bounds 区间的均值上方和下方的标准差数量。默认值为 2 。参见 extended_stats 中定义范围。    参考样例 #  以下示例创建一个以一个月为间隔的日期直方图。 sum 子聚合计算每个月的字节总和。最后， extended_stats_bucket 聚合返回这些总和的扩展统计信息："
---


# 扩展统计桶聚合

`extended_stats_bucket` 聚合是 `stats_bucket` 同级聚合的更全面的版本。除了 `stats_bucket` 提供的基本统计度量外，`extended_stats_bucket` 还计算以下指标：

- 平方和
- 方差
- 总体方差
- 抽样方差
- 标准差
- 总体标准差
- 抽样标准差
- 标准差界限：
  ** 上限
  ** 下限
  ** 种群上限
  ** 种群下限
  ** 采样上限
  ** 采样下限

标准差和方差是总体统计量；它们分别始终等于总体标准差和方差。

`std_deviation_bounds` 对象定义了一个范围，该范围在均值（默认为两个标准差）的上方和下方跨越指定的标准差数量。此对象始终包含在输出中，但仅对正态分布的数据才有意义。在解释这些值之前，请验证您的数据集是否遵循正态分布。

指定的指标必须是数值型，并且同级聚合必须是多分组聚合。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`extended_stats_bucket` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型      | 描述                                                                                                              |
| -------------- | --------- | ------------- | ----------------------------------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String        | 要聚合的聚合分组的路径。参见分组路径。                                                                            |
| `gap_policy`   | 可选      | String        | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认为 `skip` 。参见数据间隙。                          |
| `format`       | 可选      | String        | DecimalFormat 格式字符串。返回聚合的 `value_as_string` 属性中的格式化输出。                                       |
| `sigma`        | 可选      | Double 非负） | 用于计算 `std_deviation_bounds` 区间的均值上方和下方的标准差数量。默认值为 2 。参见 `extended_stats` 中定义范围。 |

## 参考样例

以下示例创建一个以一个月为间隔的日期直方图。 `sum` 子聚合计算每个月的字节总和。最后， `extended_stats_bucket` 聚合返回这些总和的扩展统计信息：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "visits_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        }
      }
    },
    "stats_monthly_bytes": {
      "extended_stats_bucket": {
        "buckets_path": "visits_per_month>sum_of_bytes",
        "sigma": 3,
        "format": "0.##E0"
      }
    }
  }
}
```

## 返回内容

响应包含所选分组的扩展统计信息。请注意，标准偏差界限适用于 3-sigma 范围；更改 sigma （或让其默认为 2 ）将返回不同的结果：

```
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "visits_per_month": {
      "buckets": [
        {
          "key_as_string": "2025-03-01T00:00:00.000Z",
          "key": 1740787200000,
          "doc_count": 480,
          "sum_of_bytes": {
            "value": 2804103
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 6849,
          "sum_of_bytes": {
            "value": 39103067
          }
        },
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 6745,
          "sum_of_bytes": {
            "value": 37818519
          }
        }
      ]
    },
    "stats_monthly_bytes": {
      "count": 3,
      "min": 2804103,
      "max": 39103067,
      "avg": 26575229.666666668,
      "sum": 79725689,
      "min_as_string": "2.8E6",
      "max_as_string": "3.91E7",
      "avg_as_string": "2.66E7",
      "sum_as_string": "7.97E7",
      "sum_of_squares": 2967153221794459,
      "variance": 282808242095406.25,
      "variance_population": 282808242095406.25,
      "variance_sampling": 424212363143109.4,
      "std_deviation": 16816903.46334325,
      "std_deviation_population": 16816903.46334325,
      "std_deviation_sampling": 20596416.2694171,
      "std_deviation_bounds": {
        "upper": 77025940.05669643,
        "lower": -23875480.72336309,
        "upper_population": 77025940.05669643,
        "lower_population": -23875480.72336309,
        "upper_sampling": 88364478.47491796,
        "lower_sampling": -35214019.141584635
      },
      "sum_of_squares_as_string": "2.97E15",
      "variance_as_string": "2.83E14",
      "variance_population_as_string": "2.83E14",
      "variance_sampling_as_string": "4.24E14",
      "std_deviation_as_string": "1.68E7",
      "std_deviation_population_as_string": "1.68E7",
      "std_deviation_sampling_as_string": "2.06E7",
      "std_deviation_bounds_as_string": {
        "upper": "7.7E7",
        "lower": "-2.39E7",
        "upper_population": "7.7E7",
        "lower_population": "-2.39E7",
        "upper_sampling": "8.84E7",
        "lower_sampling": "-3.52E7"
      }
    }
  }
}
```


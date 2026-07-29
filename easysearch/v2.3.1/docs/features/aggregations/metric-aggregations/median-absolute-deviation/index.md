---
title: "中位数绝对偏差聚合（Median Absolute Deviation）"
date: 0001-01-01
summary: "中位数绝对偏差聚合 #  median_absolute_deviation 聚合是一个单值指标聚合。中位数绝对偏差是一种变异性指标，用于衡量相对于中位数的离散程度。
相关指南（先读这些） #    聚合基础  聚合场景实践  与依赖平方误差项的标准偏差相比，中位数绝对偏差受异常值的影响较小，适用于描述非正态分布的数据。
中位数绝对偏差按以下方式计算：
median_absolute_deviation = median( | x&lt;sub&gt;i&lt;/sub&gt; - median(x&lt;sub&gt;i&lt;/sub&gt;) | )
由于内存限制，Easysearch 估计 median_absolute_deviation ，而不是直接计算它。这种估计在计算上很昂贵。您可以调整估计精度和性能之间的权衡。有关更多信息，请参阅调整估计精度。
参数说明 #  median_absolute_deviation 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 要计算中位数绝对偏差的数值字段的名称。   missing 可选 Numeric 分配给字段缺失实例的值。如果未提供，则从估计中排除具有缺失值的文档。   compression 可选 Numeric 一个调整估计精度和性能之间平衡的参数。 compression 的值必须大于 0 。默认值为 1000 。    参考样例 #  以下示例计算数据集中 DistanceMiles 字段的绝对中位数偏差："
---


# 中位数绝对偏差聚合

`median_absolute_deviation` 聚合是一个单值指标聚合。中位数绝对偏差是一种变异性指标，用于衡量相对于中位数的离散程度。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

与依赖平方误差项的标准偏差相比，中位数绝对偏差受异常值的影响较小，适用于描述非正态分布的数据。

中位数绝对偏差按以下方式计算：

`median_absolute_deviation = median( | x<sub>i</sub> - median(x<sub>i</sub>) | )`

由于内存限制，Easysearch 估计 median_absolute_deviation ，而不是直接计算它。这种估计在计算上很昂贵。您可以调整估计精度和性能之间的权衡。有关更多信息，请参阅调整估计精度。

## 参数说明

`median_absolute_deviation` 聚合采用以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| --------- | --------- | -------- | ---------------------------------------------------------------------- |
| `field` | 必需 | String | 要计算中位数绝对偏差的数值字段的名称。 |
| `missing` | 可选 | Numeric | 分配给字段缺失实例的值。如果未提供，则从估计中排除具有缺失值的文档。 |
| `compression` | 可选 | Numeric | 一个调整估计精度和性能之间平衡的参数。 `compression` 的值必须大于 0 。默认值为 `1000` 。 |

## 参考样例

以下示例计算数据集中 `DistanceMiles` 字段的绝对中位数偏差：

```
GET sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "median_absolute_deviation_DistanceMiles": {
      "median_absolute_deviation": {
        "field": "DistanceMiles"
      }
    }
  }
}
```

### 返回内容

如以下返回内容所示，聚合返回了 `median_absolute_deviation_DistanceMiles` 变量中绝对中位数偏差的估计值：

```
{
  "took": 490,
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
    "median_absolute_deviation_DistanceMiles": {
      "value": 1830.917892238693
    }
  }
}

```

## 缺省值

在计算 `median_absolute_deviation` 时会忽略缺失值和空值。你可以为聚合字段的缺失实例分配一个值。有关更多信息，请参阅缺失聚合。

## 调整估计精度

中位数绝对偏差使用 `t-digest` 数据结构进行计算，该结构采用一个 `compression` 参数来平衡性能和估计精度。 `compression` 的较低值可以提高性能，但可能会降低估计精度，如下面的请求所示：

```
GET sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "median_absolute_deviation_DistanceMiles": {
      "median_absolute_deviation": {
        "field": "DistanceMiles",
        "compression": 10
      }
    }
  }
}
```

估计误差取决于数据集，但通常低于 5%，即使对于值低至 `100` 的 `compression` 也是如此。（此处使用的低示例值 `10` 是为了说明权衡效应，并不推荐。）

请注意，在以下返回中，计算时间（ `took` 时间）有所减少，并且估计参数的值略有下降。

作为参考，Easysearch 的最佳估计（将 `compression` 任意设置得非常高）对于 `DistanceMiles` 的中值绝对偏差是 `1831.076904296875` ：

```
{
  "took": 1,
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
    "median_absolute_deviation_DistanceMiles": {
      "value": 1836.265614211182
    }
  }
}
```


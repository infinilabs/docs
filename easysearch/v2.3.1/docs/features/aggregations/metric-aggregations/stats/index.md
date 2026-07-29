---
title: "统计聚合（Stats）"
date: 0001-01-01
summary: "统计聚合 #  stats 聚合是一个多值指标聚合，用于计算数值数据的汇总。这种聚合有助于快速了解数值字段的分布情况。它可以直接作用于字段，应用脚本来派生值，或处理缺少字段的文档。stats 聚合返回五个值：
相关指南（先读这些） #     聚合基础
   聚合场景实践
  count : 收集到的值的数量
  min : 最低值
  max : 最高值
  sum : 所有值的总和
  avg : 值的平均数（总和除以数量）
  参数说明 #  stats 聚合支持以下可选参数。
   参数 必需/可选 数据类型 描述     field 必需 String 要聚合的字段。必须是数值字段。   script 可选 Object 用于计算聚合自定义值的脚本。可单独使用或与 field 一起使用。   missing 可选 Number 用于缺少目标字段的文档的默认值。    参考样例 #  以下示例计算 stats 聚合的电力使用情况。"
---


# 统计聚合

`stats` 聚合是一个多值指标聚合，用于计算数值数据的汇总。这种聚合有助于快速了解数值字段的分布情况。它可以直接作用于字段，应用脚本来派生值，或处理缺少字段的文档。`stats` 聚合返回五个值：

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

- `count` : 收集到的值的数量
- `min` : 最低值
- `max` : 最高值
- `sum` : 所有值的总和
- `avg` : 值的平均数（总和除以数量）

## 参数说明

stats 聚合支持以下可选参数。

| 参数      | 必需/可选 | 数据类型 | 描述                                                        |
| --------- | --------- | -------- | ----------------------------------------------------------- |
| `field`   | 必需      | String   | 要聚合的字段。必须是数值字段。                              |
| `script`  | 可选      | Object   | 用于计算聚合自定义值的脚本。可单独使用或与 field 一起使用。 |
| `missing` | 可选      | Number   | 用于缺少目标字段的文档的默认值。                            |

## 参考样例

以下示例计算 `stats` 聚合的电力使用情况。

创建一个名为 `power_usage` 的索引，并添加包含给定小时内消耗的千瓦时 (kWh) 数量的文档：

```
PUT /power_usage/_bulk?refresh=true
{"index": {}}
{"device_id": "A1", "kwh": 1.2}
{"index": {}}
{"device_id": "A2", "kwh": 0.7}
{"index": {}}
{"device_id": "A3", "kwh": 1.5}
```

要在所有文档中对 kwh 字段计算统计信息，使用一个名为 `consumption_stats` 的 `stats` 聚合，聚合字段为 `kwh` 。将 `size` 设置为 0 表示不应返回文档命中：

```
GET /power_usage/_search
{
  "size": 0,
  "aggs": {
    "consumption_stats": {
      "stats": {
        "field": "kwh"
      }
    }
  }
}
```

返回内容为索引中的三个文档包含 `count` 、 `min` 、 `max` 、 `avg` 和 `sum` 值：

```
{
  ...
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "consumption_stats": {
      "count": 3,
      "min": 0.699999988079071,
      "max": 1.5,
      "avg": 1.1333333452542622,
      "sum": 3.400000035762787
    }
  }
}
```

### 每个分组运行 stats 聚合

您可以通过在 `device_id` 字段中将 `stats` 聚合嵌套在 `terms` 聚合中来为每个设备计算单独的统计信息。 `terms` 聚合根据唯一的 `device_id` 值将文档分组，而 `stats` 聚合在每个分组内计算汇总统计信息：

```
GET /power_usage/_search
{
  "size": 0,
  "aggs": {
    "per_device": {
      "terms": {
        "field": "device_id.keyword"
      },
      "aggs": {
        "device_usage_stats": {
          "stats": {
            "field": "kwh"
          }
        }
      }
    }
  }
}
```

返回为每个 `device_id` 返回一个分组，每个分组内包含计算出的 `count` 、 `min` 、 `max` 、 `avg` 和 `sum` 字段：

```
{
  ...
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "per_device": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "A1",
          "doc_count": 1,
          "device_usage_stats": {
            "count": 1,
            "min": 1.2000000476837158,
            "max": 1.2000000476837158,
            "avg": 1.2000000476837158,
            "sum": 1.2000000476837158
          }
        },
        {
          "key": "A2",
          "doc_count": 1,
          "device_usage_stats": {
            "count": 1,
            "min": 0.699999988079071,
            "max": 0.699999988079071,
            "avg": 0.699999988079071,
            "sum": 0.699999988079071
          }
        },
        {
          "key": "A3",
          "doc_count": 1,
          "device_usage_stats": {
            "count": 1,
            "min": 1.5,
            "max": 1.5,
            "avg": 1.5,
            "sum": 1.5
          }
        }
      ]
    }
  }
}

```

这使您能够通过单个查询比较不同设备的使用统计信息。

### 使用脚本计算派生值

您也可以使用脚本计算 `stats` 聚合中使用的值。当指标来自文档字段或需要转换时，这很有用。

例如，在运行 `stats` 聚合之前，将千瓦时（kWh）转换为瓦时（Wh），因为 1 kWh 等于 1,000 Wh ，你可以使用一个将每个值乘以 1,000 的脚本。以下脚本 `doc['kwh'].value * 1000` 用于推导每个文档的输入值：

```
GET /power_usage/_search
{
  "size": 0,
  "aggs": {
    "usage_wh_stats": {
      "stats": {
        "script": {
          "source": "doc['kwh'].value * 1000"
        }
      }
    }
  }
}

```

返回的 `stats` 聚合反映了 1200 、 700 和 1500 Wh 的值

```
{
  ...
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "usage_wh_stats": {
      "count": 3,
      "min": 699.999988079071,
      "max": 1500,
      "avg": 1133.3333452542622,
      "sum": 3400.000035762787
    }
  }
}
```

### 使用带有字段的值脚本

当将字段与转换结合使用时，你可以同时指定 `field` 和 `script` 。这允许使用 `_value` 变量来在脚本中引用字段的值。

以下示例在计算 `stats` 聚合之前将每个能量读数增加 5%：

```
GET /power_usage/_search
{
  "size": 0,
  "aggs": {
    "adjusted_usage": {
      "stats": {
        "field": "kwh",
        "script": {
          "source": "_value * 1.05"
        }
      }
    }
  }
}

```

### 缺省值

如果某些文档不包含目标字段，它们默认会被排除在聚合之外。要使用默认值包含它们，你可以指定 `missing` 参数。

以下请求将缺失的 kwh 值视为 0.0 ：

```
GET /power_usage/_search
{
  "size": 0,
  "aggs": {
    "consumption_with_default": {
      "stats": {
        "field": "kwh",
        "missing": 0.0
      }
    }
  }
}

```


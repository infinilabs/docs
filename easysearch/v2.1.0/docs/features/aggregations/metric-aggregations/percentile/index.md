---
title: "百分位数聚合（Percentiles）"
date: 0001-01-01
summary: "百分位数聚合 #  percentiles 聚合估计数值字段在给定百分位处的值。这对于理解分布边界很有用。
例如，load_time 的 95th 百分位 = 120ms 表示 95% 的值小于或等于 120 毫秒。
与 cardinality 指标类似，percentiles 指标也是近似的。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  percentiles 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 用于计算百分位数的数值字段。   percents 可选 Array of doubles 返回百分位数列表。默认为 [1, 5, 25, 50, 75, 95, 99] 。   keyed 可选 Boolean 如果设置为 false ，则将结果作为数组返回。否则将结果作为 JSON 对象返回。默认值为 true 。   tdigest."
---


# 百分位数聚合

`percentiles` 聚合估计数值字段在给定百分位处的值。这对于理解分布边界很有用。

例如，`load_time` 的 95th 百分位 = `120ms` 表示 95% 的值小于或等于 120 毫秒。

与 `cardinality` 指标类似，`percentiles` 指标也是近似的。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`percentiles` 聚合采用以下参数。

| 参数                                     | 必需/可选 | 数据类型         | 描述                                                                                    |
| ---------------------------------------- | --------- | ---------------- | --------------------------------------------------------------------------------------- |
| `field`                                  | 必需      | String           | 用于计算百分位数的数值字段。                                                            |
| `percents`                               | 可选      | Array of doubles | 返回百分位数列表。默认为 [1, 5, 25, 50, 75, 95, 99] 。                                  |
| `keyed`                                  | 可选      | Boolean          | 如果设置为 false ，则将结果作为数组返回。否则将结果作为 JSON 对象返回。默认值为 true 。 |
| `tdigest.compression`                    | 可选      | Double           | 控制 `tdigest` 算法的准确性和内存使用。参见使用 `tdigest` 进行精度调整。                |
| `hdr.number_of_significant_value_digits` | 可选      | Integer          | HDR 直方图的精度设置。参见 HDR 直方图。                                                 |
| `missing`                                | 可选      | Numeric          | 当文档中目标字段缺失时使用的默认值。                                                    |
| `script`                                 | 可选      | Object           | 用于计算自定义值而不是使用字段的脚本。支持内联和存储脚本。                              |

## 参考样例

首先，创建一个索引：

```
PUT /latency_data
{
  "mappings": {
    "properties": {
      "load_time": {
        "type": "double"
      }
    }
  }
}

```

添加示例数值以说明百分位数计算：

```
POST /latency_data/_bulk
{ "index": {} }
{ "load_time": 20 }
{ "index": {} }
{ "load_time": 40 }
{ "index": {} }
{ "load_time": 60 }
{ "index": {} }
{ "load_time": 80 }
{ "index": {} }
{ "load_time": 100 }
{ "index": {} }
{ "load_time": 120 }
{ "index": {} }
{ "load_time": 140 }
```

### 百分位数聚合

以下示例计算 load_time 字段的默认百分位数集：

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "load_time_percentiles": {
      "percentiles": {
        "field": "load_time"
      }
    }
  }
}
```

默认情况下，会返回第 1、5、25、50、75、95 和 99 个百分位数：

```
{
  ...
  "aggregations": {
    "load_time_percentiles": {
      "values": {
        "1.0": 20,
        "5.0": 20,
        "25.0": 40,
        "50.0": 80,
        "75.0": 120,
        "95.0": 140,
        "99.0": 140
      }
    }
  }
}
```

## 自定义分位数

您可以使用 percents 数组指定确切的百分位数：

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "load_time_percentiles": {
      "percentiles": {
        "field": "load_time",
        "percents": [50, 90, 99]
      }
    }
  }
}

```

返回内容仅包含所请求的三个百分位聚合：

```
{
  ...
  "aggregations": {
    "load_time_percentiles": {
      "values": {
        "50.0": 80,
        "90.0": 140,
        "99.0": 140
      }
    }
  }
}
```

### 键值响应

可以通过将 `keyed` 参数设置为 `false` 将返回的聚合格式从 JSON 对象更改为键值对列表：

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "load_time_percentiles": {
      "percentiles": {
        "field": "load_time",
        "keyed": false
      }
    }
  }
}
```

响应以值数组的形式提供百分位数：

```
{
  ...
  "aggregations": {
    "load_time_percentiles": {
      "values": [
        {
          "key": 1,
          "value": 20
        },
        {
          "key": 5,
          "value": 20
        },
        {
          "key": 25,
          "value": 40
        },
        {
          "key": 50,
          "value": 80
        },
        {
          "key": 75,
          "value": 120
        },
        {
          "key": 95,
          "value": 140
        },
        {
          "key": 99,
          "value": 140
        }
      ]
    }
  }
}

```

### 使用 tdigest 进行精确度调整

`tdigest` 算法是计算百分位数的默认方法。它提供了一种内存高效的方式来估计百分位数排名，尤其是在处理浮点数据（如响应时间或延迟）时。

与精确的百分位数计算不同， `tdigest` 使用概率方法将值分组到质心——即总结分布的小型簇。这种方法能够在无需将所有原始数据存储在内存中的情况下，为大多数百分位数提供准确的估计。

该算法设计为在分布的尾部附近（即低百分位数（如第 1 位）和高百分位数（如第 99 位））具有高度准确性，这些通常是性能分析中最重要的一部分。您可以使用 `compression` 参数控制结果的精度。

较高的 `compression` 值意味着使用更多的质心，这会增加准确性（尤其是在尾部），但需要更多的内存和 CPU。较低的 `compression` 值会减少内存使用并加快执行速度，但结果可能不够准确。

适合使用 `tdigest` 的情况：

- 您的数据包含浮点值，例如响应时间、延迟或持续时间。
- 您需要在极端百分位数中获得准确的结果，例如第 1 位或第 99 位。

避免使用 `tdigest` 的情况：

- 您只处理整数数据，并希望获得最大速度。
- 您不太在意分布尾部的准确性，更倾向于快速聚合（可以考虑使用 hdr 代替）。

以下示例将 `tdigest.compression` 设置为 200 :

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "load_time_percentiles": {
      "percentiles": {
        "field": "load_time",
        "tdigest": {
          "compression": 200
        }
      }
    }
  }
}

```

### HDR 直方图

高动态范围（HDR）直方图是计算百分位的另一种方法。它特别适用于处理大型数据集和延迟测量。它专为速度而设计，同时支持宽动态范围值，并保持固定且可配置的精度水平。

与 `tdigest` 不同，后者在分布的尾部（极端百分位）提供更高的准确性，HDR 优先考虑速度和范围内均匀的准确性。当分组的数量很大且不需要对稀有值进行极端精确度时，它效果最佳。

例如，如果你正在测量从 1 微秒到 1 小时的范围内的响应时间，并使用 3 位有效数字配置 HDR，它将记录值精度为 ±1 微秒（直到 1 毫秒）和 ±3.6 秒（接近 1 小时）。

这种权衡使得 HDR 比 `tdigest` 快得多，但内存消耗更大。

下表展示了 HDR 显著数字的分解情况。
|有效数字|相对精度（最大误差）|
| --- | --- |
|1 |1 份在 10 份中 = 10%|
|2 |1 份在 100 份中 = 1%|
|3 |1 份在 1000 中 = 0.1%|
|4 |1 份在 10000 中 = 0.01%|
|5 |1 份在 100000 中 = 0.001%|

如果您需要使用 HDR，则应：

- 正在跨多个分组进行聚合。
- 不需要在尾部的百分位数上要求极端的精确度。
- 确保有足够的内存可用。

你应该避免 HDR，如果：

- 尾部精度很重要。
- 你正在分析偏斜或稀疏的数据分布。

以下示例中将 `hdr.number_of_significant_value_digits` 设置为 3 ：

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "load_time_percentiles": {
      "percentiles": {
        "field": "load_time",
        "hdr": {
          "number_of_significant_value_digits": 3
        }
      }
    }
  }
}
```

### 缺省值

使用 missing 设置为不包含目标字段的文档配置备用值：

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "load_time_percentiles": {
      "percentiles": {
        "field": "load_time",
        "missing": 0
      }
    }
  }
}

```

## Script 脚本

可以使用脚本动态计算值，而不是指定字段。当您需要应用转换（如货币转换或应用权重）时，这很有用。

### 内联脚本

使用脚本计算派生值：

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "adjusted_percentiles": {
      "percentiles": {
        "script": {
          "source": "doc['load_time'].value * 1.2"
        },
        "percents": [50, 95]
      }
    }
  }
}

```

### 存储脚本

首先，使用以下请求创建一个示例脚本：

```
POST _scripts/load_script
{
  "script": {
    "lang": "painless",
    "source": "doc[params.field].value * params.multiplier"
  }
}
```

然后用 `percentiles` 聚合中的存储脚本，提供 `params` 存储脚本所需的：

```
GET /latency_data/_search
{
  "size": 0,
  "aggs": {
    "adjusted_percentiles": {
      "percentiles": {
        "script": {
          "id": "load_script",
          "params": {
            "field": "load_time",
            "multiplier": 1.2
          }
        },
        "percents": [50, 95]
      }
    }
  }
}
```


---
title: "百分位排名聚合（Percentile Ranks）"
date: 0001-01-01
summary: "百分位排名聚合 #  percentile_ranks 聚合估计低于或等于给定阈值的观测值百分比。这对于了解特定值在值分布中的相对位置很有用。
例如，您可以使用百分位排名聚合来学习交易金额 45 与数据集中其他交易值相比如何。百分位排名聚合返回一个值，如 82.3，这意味着 82.3% 的交易额低于或等于 45。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  percentile_ranks 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 用于计算百分位数的数值字段。   values 必需 Array of doubles 用于计算百分位数的值。   keyed 可选 Boolean 如果设置为 false ，则将结果作为数组返回。否则将结果作为 JSON 对象返回。默认值为 true 。   tdigest.compression 可选 Double 控制 tdigest 算法的准确性和内存使用。参见使用 tdigest 进行精度调整。   hdr."
---


# 百分位排名聚合

`percentile_ranks` 聚合估计低于或等于给定阈值的观测值百分比。这对于了解特定值在值分布中的相对位置很有用。

例如，您可以使用百分位排名聚合来学习交易金额 `45` 与数据集中其他交易值相比如何。百分位排名聚合返回一个值，如 `82.3`，这意味着 82.3% 的交易额低于或等于 `45`。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`percentile_ranks` 聚合采用以下参数。

| 参数                                     | 必需/可选 | 数据类型         | 描述                                                                                    |
| ---------------------------------------- | --------- | ---------------- | --------------------------------------------------------------------------------------- |
| `field`                                  | 必需      | String           | 用于计算百分位数的数值字段。                                                            |
| `values`                                 | 必需      | Array of doubles | 用于计算百分位数的值。                                                                  |
| `keyed`                                  | 可选      | Boolean          | 如果设置为 false ，则将结果作为数组返回。否则将结果作为 JSON 对象返回。默认值为 true 。 |
| `tdigest.compression`                    | 可选      | Double           | 控制 `tdigest` 算法的准确性和内存使用。参见使用 `tdigest` 进行精度调整。                |
| `hdr.number_of_significant_value_digits` | 可选      | Integer          | HDR 直方图的精度设置。参见 HDR 直方图。                                                 |
| `missing`                                | 可选      | Numeric          | 当文档中目标字段缺失时使用的默认值。                                                    |
| `script`                                 | 可选      | Object           | 用于计算自定义值而不是使用字段的脚本。支持内联和存储脚本。                              |

## 参考样例

首先，创建一个示例索引：

```
PUT /transaction_data
{
  "mappings": {
    "properties": {
      "amount": {
        "type": "double"
      }
    }
  }
}
```

添加示例数值以说明百分位数排名计算：

```
POST /transaction_data/_bulk
{ "index": {} }
{ "amount": 10 }
{ "index": {} }
{ "amount": 20 }
{ "index": {} }
{ "amount": 30 }
{ "index": {} }
{ "amount": 40 }
{ "index": {} }
{ "amount": 50 }
{ "index": {} }
{ "amount": 60 }
{ "index": {} }
{ "amount": 70 }
```

运行 `percentile_ranks` 聚合来计算某些值与整体分布的比较情况：

```
GET /transaction_data/_search
{
  "size": 0,
  "aggs": {
    "rank_check": {
      "percentile_ranks": {
        "field": "amount",
        "values": [25, 55]
      }
    }
  }
}
```

表明 28.6%的值小于或等于 25 ，71.4%的值小于或等于 55 ：

```
{
  ...
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "rank_check": {
      "values": {
        "25.0": 28.57142857142857,
        "55.0": 71.42857142857143
      }
    }
  }
}

```

## 键值

可以通过将 `keyed` 参数设置为 `false` 来更改返回的聚合格式，从 JSON 对象更改为键值对列表：

```
GET /transaction_data/_search
{
  "size": 0,
  "aggs": {
    "rank_check": {
      "percentile_ranks": {
        "field": "amount",
        "values": [25, 55],
        "keyed": false
      }
    }
  }
}
```

返回内容包含一个数组而不是对象：

```
{
  ...
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "rank_check": {
      "values": [
        {
          "key": 25,
          "value": 28.57142857142857
        },
        {
          "key": 55,
          "value": 71.42857142857143
        }
      ]
    }
  }
}
```

## 使用 tdigest 进行精确度调整

默认情况下，百分位数排名使用 `tdigest` 算法计算。您可以通过指定 `tdigest.compression` 参数来控制准确性和内存使用之间的权衡。更高的值提供更好的准确性，但需要更多的内存。有关 `tdigest` 工作原理的更多信息，请参阅使用 `tdigest` 进行精度调整。
以下示例中将 `tdigest.compression` 设置为 `200` ：

```
GET /transaction_data/_search
{
  "size": 0,
  "aggs": {
    "rank_check": {
      "percentile_ranks": {
        "field": "amount",
        "values": [25, 55],
        "tdigest": {
          "compression": 200
        }
      }
    }
  }
}

```

### HDR 直方图

作为 `tdigest` 的替代方案，您可以使用高动态范围（HDR）直方图算法，该算法更适合大量分组和快速处理。有关 HDR 直方图的工作原理的更多信息，请参阅 HDR 直方图。

如果您需要使用 HDR，则应：

- 正在跨多个分组进行聚合。
- 不需要在尾部的百分位数上要求极端的精确度。
- 确保有足够的内存可用。

你应该避免 HDR，如果：

- 尾部精度很重要。
- 你正在分析偏斜或稀疏的数据分布。

以下示例中将 `hdr.number_of_significant_value_digits` 设置为 3 ：

```
GET /transaction_data/_search
{
  "size": 0,
  "aggs": {
    "rank_check": {
      "percentile_ranks": {
        "field": "amount",
        "values": [25, 55],
        "hdr": {
          "number_of_significant_value_digits": 3
        }
      }
    }
  }
}
```

## 缺省值

如果某些文档缺少目标字段，你可以通过设置 `missing` 参数来指示查询使用备用值。以下示例确保没有 `amount` 字段的文档被视为其值为 0 ，并包含在百分位数计算中：

```
GET /transaction_data/_search
{
  "size": 0,
  "aggs": {
    "rank_check": {
      "percentile_ranks": {
        "field": "amount",
        "values": [25, 55],
        "missing": 0
      }
    }
  }
}
```

### Script 脚本

除了指定字段，您还可以使用脚本动态计算值。这适用于需要应用转换的情况，例如货币转换或应用权重。

#### 内联脚本

以下示例使用内联脚本计算转换后的值 `30` 和 `60` 与增加 10%的 `amount` 字段值的百分位数排名：

```
GET /transaction_data/_search
{
  "size": 0,
  "aggs": {
    "rank_check": {
      "percentile_ranks": {
        "values": [30, 60],
        "script": {
          "source": "doc['amount'].value * 1.1"
        }
      }
    }
  }
}

```

#### 存储脚本

要使用存储脚本，首先使用以下请求创建它：

```
POST _scripts/percentile_script
{
  "script": {
    "lang": "painless",
    "source": "doc[params.field].value * params.multiplier"
  }
}

```

然后用 `percentile_ranks` 聚合中的存储脚本：

```
GET /transaction_data/_search
{
  "size": 0,
  "aggs": {
    "rank_check": {
      "percentile_ranks": {
        "values": [30, 60],
        "script": {
          "id": "percentile_script",
          "params": {
            "field": "amount",
            "multiplier": 1.1
          }
        }
      }
    }
  }
}

```


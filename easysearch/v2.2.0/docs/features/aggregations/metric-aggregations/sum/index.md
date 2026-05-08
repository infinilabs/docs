---
title: "求和聚合（Sum）"
date: 0001-01-01
summary: "求和聚合 #  sum 聚合是一种单值指标聚合，计算字段中匹配文档中提取的数值的总和。此聚合常用于计算诸如收入、数量或持续时间等指标的总计。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  sum 聚合接受以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 聚合的字段。必须是数值字段。   script 可选 Object 用于计算聚合自定义值的脚本。可以替代或与 field 结合使用。   missing 可选 Number 缺少目标字段时使用的默认值。    参考样例 #  以下示例演示了如何计算物流索引中记录的交付总重量。
创建一个索引：
PUT /deliveries { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;shipment_id&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;weight_kg&#34;: { &#34;type&#34;: &#34;double&#34; } } } } 添加示例文档："
---


# 求和聚合

`sum` 聚合是一种单值指标聚合，计算字段中匹配文档中提取的数值的总和。此聚合常用于计算诸如收入、数量或持续时间等指标的总计。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`sum` 聚合接受以下参数。

| 参数      | 必需/可选 | 数据类型 | 描述                                                      |
| --------- | --------- | -------- | --------------------------------------------------------- |
| `field`   | 必需      | String   | 聚合的字段。必须是数值字段。                              |
| `script`  | 可选      | Object   | 用于计算聚合自定义值的脚本。可以替代或与 field 结合使用。 |
| `missing` | 可选      | Number   | 缺少目标字段时使用的默认值。                              |

## 参考样例

以下示例演示了如何计算物流索引中记录的交付总重量。

创建一个索引：

```
PUT /deliveries
{
  "mappings": {
    "properties": {
      "shipment_id": { "type": "keyword" },
      "weight_kg": { "type": "double" }
    }
  }
}

```

添加示例文档：

```
POST /deliveries/_bulk?refresh=true
{"index": {}}
{"shipment_id": "S001", "weight_kg": 12.5}
{"index": {}}
{"shipment_id": "S002", "weight_kg": 7.8}
{"index": {}}
{"shipment_id": "S003", "weight_kg": 15.0}
{"index": {}}
{"shipment_id": "S004", "weight_kg": 10.3}
```

以下请求计算 `deliveries` 索引中所有文档的总权重，通过将 `size` 设置为 0 来忽略文档命中，并返回 `weight_kg` 的总和：

```
GET /deliveries/_search
{
  "size": 0,
  "aggs": {
    "total_weight": {
      "sum": {
        "field": "weight_kg"
      }
    }
  }
}

```

返回包含值 `45.6` ，对应于 `12.5 + 7.8 + 15.0 + 10.3` 的总和：

```
{
  ...
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "total_weight": {
      "value": 45.6
    }
  }
}
```

### 使用脚本来计算值

你可以提供一个脚本来计算聚合中的值，而不是直接指定一个字段。这在值必须经过推导或调整时非常有用。

在以下示例中，每个重量在使用脚本求和之前都会从千克转换为克：

```
GET /deliveries/_search
{
  "size": 0,
  "aggs": {
    "total_weight_grams": {
      "sum": {
        "script": {
          "source": "doc['weight_kg'].value * 1000"
        }
      }
    }
  }
}

```

内容中 `total_weight_grams` 为 45600 ：

```
{
  ...
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "total_weight_grams": {
      "value": 45600
    }
  }
}
```

### 将字段与值脚本结合使用

你也可以同时指定 `field` 和 `script` ，使用特殊变量 `_value` 来引用字段的值。这在对现有字段值应用转换时非常有用。

以下示例在求和之前将所有权重增加 10%：

```
GET /deliveries/_search
{
  "size": 0,
  "aggs": {
    "adjusted_weight": {
      "sum": {
        "field": "weight_kg",
        "script": {
          "source": "Math.round(_value * 110) / 100.0"
        }
      }
    }
  }
}

```

反映了原始总重量增加了 10%：

```
{
  ...
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "adjusted_weight": {
      "value": 50.16
    }
  }
}
```

### 缺省值

缺少目标字段的文档默认会被忽略。要使用默认值包含它们，请使用 `missing` 参数。

以下示例将默认值 0 分配给缺少的 `weight_kg` 字段。这确保了缺少该字段的文档被视为 `weight_kg` 设置为 0 并包含在聚合中。

```
GET /deliveries/_search
{
  "size": 0,
  "aggs": {
    "total_weight_with_missing": {
      "sum": {
        "field": "weight_kg",
        "missing": 0
      }
    }
  }
}

```


---
title: "值计数聚合（Value Count）"
date: 0001-01-01
summary: "值计数聚合 #  value_count 聚合是一个单值指标聚合，用于计算聚合所基于的值的数量。
例如，您可以将 value_count 指标与 avg 指标一起使用来查找聚合使用多少个数字来计算平均值。
相关指南（先读这些） #    聚合基础  聚合场景实践  GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;number_of_values&#34;: { &#34;value_count&#34;: { &#34;field&#34;: &#34;taxful_total_price&#34; } } } } 返回内容
... &#34;aggregations&#34; : { &#34;number_of_values&#34; : { &#34;value&#34; : 4675 } } } 参数说明 #     参数 必需/可选 数据类型 描述     field 必填 String 要计数的字段名称。   script 可选 Object 使用脚本生成要计数的值，替代 field。   missing 可选 任意 为缺失该字段的文档提供默认值。    value_count vs."
---


# 值计数聚合

`value_count` 聚合是一个单值指标聚合，用于计算聚合所基于的值的数量。

例如，您可以将 `value_count` 指标与 `avg` 指标一起使用来查找聚合使用多少个数字来计算平均值。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

```
GET sample_data_ecommerce/_search
{
  "size": 0,
   "aggs": {
    "number_of_values": {
      "value_count": {
        "field": "taxful_total_price"
      }
    }
  }
}

```

返回内容

```
...
  "aggregations" : {
    "number_of_values" : {
      "value" : 4675
    }
  }
}
```

## 参数说明

| 参数     | 必需/可选 | 数据类型 | 描述                                           |
| -------- | --------- | -------- | ---------------------------------------------- |
| `field`  | 必填      | String   | 要计数的字段名称。                              |
| `script` | 可选      | Object   | 使用脚本生成要计数的值，替代 `field`。           |
| `missing`| 可选      | 任意     | 为缺失该字段的文档提供默认值。                   |

## value_count vs. cardinality

`value_count` 统计的是字段值的 **总出现次数**（包括重复值），而 `cardinality` 统计的是 **唯一值** 的数量：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "total_values": {
      "value_count": { "field": "customer_id" }
    },
    "unique_customers": {
      "cardinality": { "field": "customer_id" }
    }
  }
}
```

| 聚合            | 含义                           | 示例结果          |
| --------------- | ------------------------------ | ----------------- |
| `value_count`   | 字段值总出现次数（含重复）     | 4675（总订单数）  |
| `cardinality`   | 字段唯一值数量（去重）         | 46（独立客户数）  |

## 使用脚本

可以用脚本组合多个字段的值来计数：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "product_count": {
      "value_count": {
        "script": {
          "source": "doc['products.product_id'].size()"
        }
      }
    }
  }
}
```


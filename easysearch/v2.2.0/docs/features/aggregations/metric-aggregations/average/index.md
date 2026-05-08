---
title: "平均值聚合（Avg）"
date: 0001-01-01
summary: "平均值聚合 #  avg 聚合是一个单值指标聚合，它返回某个字段的平均值。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  avg 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 计算平均值的字段。   missing 可选 Float 要分配给字段缺失实例的值。默认情况下， avg 会在计算中忽略缺失值。    参考样例 #  以下示例请求计算示例数据中 taxful_total_price 字段的平均值：
GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;avg_taxful_total_price&#34;: { &#34;avg&#34;: { &#34;field&#34;: &#34;taxful_total_price&#34; } } } } 返回内容 #  返回内容包含 taxful_total_price 的平均值："
---


# 平均值聚合

`avg` 聚合是一个单值指标聚合，它返回某个字段的平均值。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`avg` 聚合采用以下参数。

| 参数      | 必需/可选 | 数据类型 | 描述                                                                |
| --------- | --------- | -------- | ------------------------------------------------------------------- |
| `field`   | 必需      | String   | 计算平均值的字段。                                                  |
| `missing` | 可选      | Float    | 要分配给字段缺失实例的值。默认情况下， `avg` 会在计算中忽略缺失值。 |

## 参考样例

以下示例请求计算示例数据中 `taxful_total_price` 字段的平均值：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "avg_taxful_total_price": {
      "avg": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

## 返回内容

返回内容包含 `taxful_total_price` 的平均值：

```
{
  "took": 85,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "avg_taxful_total_price": {
      "value": 75.05542864304813
    }
  }
}
```

可以使用聚合名称 `avg_taxful_total_price` 作为从聚合获取结果的键名。

## 缺失值处理

通过提取以下文档来准备示例索引。请注意，第二个文档缺少 `gpa` 值：

```
POST _bulk
{ "create": { "_index": "students", "_id": "1" } }
{ "name": "John Doe", "gpa": 3.89, "grad_year": 2022}
{ "create": { "_index": "students", "_id": "2" } }
{ "name": "Jonathan Powers", "grad_year": 2025 }
{ "create": { "_index": "students", "_id": "3" } }
{ "name": "Jane Doe", "gpa": 3.52, "grad_year": 2024 }
```

### 示例：替换缺失值

取平均值，将缺失的 GPA 字段替换为 0 ：

```
GET students/_search
{
  "size": 0,
  "aggs": {
    "avg_gpa": {
      "avg": {
        "field": "gpa",
        "missing": 0
      }
    }
  }
}

```

返回内容如下。可以与下一个忽略了缺失值的示例做个比较：

```
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "avg_gpa": {
      "value": 2.4700000286102295
    }
  }
}
```

### 示例：忽略缺失值

取平均值但不分配 `missing` 参数：

```
GET students/_search
{
  "size": 0,
  "aggs": {
    "avg_gpa": {
      "avg": {
        "field": "gpa"
      }
    }
  }
}

```

聚合器计算平均值，省略包含缺失字段值的文档（默认行为）：

```
{
  "took": 255,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "avg_gpa": {
      "value": 3.7050000429153442
    }
  }
}
```


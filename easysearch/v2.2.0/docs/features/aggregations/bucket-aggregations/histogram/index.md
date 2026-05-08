---
title: "直方图聚合（Histogram）"
date: 0001-01-01
summary: "直方图聚合 #  histogram 聚合根据指定的间隔对文档进行分组。
使用 histogram 聚合，您可以非常轻松地可视化给定范围内文档中值的分布。
相关指南（先读这些） #    聚合基础  聚合场景实践  以下示例将 number_of_bytes 字段按 10,000 个间隔进行分组：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;number_of_bytes&#34;: { &#34;histogram&#34;: { &#34;field&#34;: &#34;bytes&#34;, &#34;interval&#34;: 10000 } } } } 返回内容
... &#34;aggregations&#34; : { &#34;number_of_bytes&#34; : { &#34;buckets&#34; : [ { &#34;key&#34; : 0.0, &#34;doc_count&#34; : 13372 }, { &#34;key&#34; : 10000.0, &#34;doc_count&#34; : 702 } ] } } 参数说明 #  histogram 聚合支持以下参数。"
---


# 直方图聚合

`histogram` 聚合根据指定的间隔对文档进行分组。

使用 `histogram` 聚合，您可以非常轻松地可视化给定范围内文档中值的分布。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

以下示例将 number_of_bytes 字段按 10,000 个间隔进行分组：

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
    }
  }
}
```

返回内容

```
...
"aggregations" : {
  "number_of_bytes" : {
    "buckets" : [
      {
        "key" : 0.0,
        "doc_count" : 13372
      },
      {
        "key" : 10000.0,
        "doc_count" : 702
      }
    ]
  }
 }
```

## 参数说明

histogram 聚合支持以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| ------------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------ |
| `interval	` | 必填 | Numeric | 构造每个分组所使用的字段值宽度。|
| `min_doc_count` | 可选 | Integer | 桶中至少需要包含的文档数量才会出现在结果中。默认为 0，设为 1 可隐藏空桶。|
| `offset` | 可选 | Numeric | 桶边界的偏移量。例如 interval=10，offset=5 时桶为 [5,15)、[15,25)...。默认为 0。|
| `extended_bounds` | 可选 | Object | 包含 `min` 和 `max`，强制直方图覆盖指定范围（即使没有数据）。不会过滤文档。|
| `hard_bounds` | 可选 | Object | 包含 `min` 和 `max`，限制直方图的范围，超出范围的桶会被丢弃。|
| `order` | 可选 | Object | 桶的排序方式。默认按 `key` 升序。|
| `keyed` | 可选 | Boolean | 若为 `true`，以键值对而非数组形式返回结果。默认为 `false`。|

### 显示空桶

设置 `min_doc_count` 为 `0` 会返回所有桶（包括没有文档的桶），配合 `extended_bounds` 可以强制覆盖完整范围：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "bytes_distribution": {
      "histogram": {
        "field": "bytes",
        "interval": 5000,
        "min_doc_count": 0,
        "extended_bounds": {
          "min": 0,
          "max": 20000
        }
      }
    }
  }
}
```

### 嵌套子聚合

直方图聚合的每个桶内都可以嵌套子聚合。例如，查看每个字节范围桶中的平均响应时间：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "bytes_ranges": {
      "histogram": {
        "field": "bytes",
        "interval": 5000
      },
      "aggs": {
        "avg_response": {
          "avg": {
            "field": "response_time"
          }
        }
      }
    }
  }
}
```


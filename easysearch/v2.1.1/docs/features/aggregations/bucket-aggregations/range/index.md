---
title: "范围聚合（Range）"
date: 0001-01-01
summary: "范围聚合 #  range 聚合允许你为每个分组定义范围。
相关指南（先读这些） #    聚合基础  聚合场景实践  例如，你可以找到在 1000 和 2000 之间、2000 和 3000 之间以及 3000 和 4000 之间的字节数。在 range 参数中，你可以将范围定义为数组对象。
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;number_of_bytes_distribution&#34;: { &#34;range&#34;: { &#34;field&#34;: &#34;bytes&#34;, &#34;ranges&#34;: [ { &#34;from&#34;: 1000, &#34;to&#34;: 2000 }, { &#34;from&#34;: 2000, &#34;to&#34;: 3000 }, { &#34;from&#34;: 3000, &#34;to&#34;: 4000 } ] } } } } 响应包含 from 键值，并排除 to 键值："
---


# 范围聚合

`range` 聚合允许你为每个分组定义范围。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

例如，你可以找到在 1000 和 2000 之间、2000 和 3000 之间以及 3000 和 4000 之间的字节数。在 `range` 参数中，你可以将范围定义为数组对象。

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes_distribution": {
      "range": {
        "field": "bytes",
        "ranges": [
          {
            "from": 1000,
            "to": 2000
          },
          {
            "from": 2000,
            "to": 3000
          },
          {
            "from": 3000,
            "to": 4000
          }
        ]
      }
    }
  }
}
```

响应包含 `from` 键值，并排除 `to` 键值：

```
...
"aggregations" : {
  "number_of_bytes_distribution" : {
    "buckets" : [
      {
        "key" : "1000.0-2000.0",
        "from" : 1000.0,
        "to" : 2000.0,
        "doc_count" : 805
      },
      {
        "key" : "2000.0-3000.0",
        "from" : 2000.0,
        "to" : 3000.0,
        "doc_count" : 1369
      },
      {
        "key" : "3000.0-4000.0",
        "from" : 3000.0,
        "to" : 4000.0,
        "doc_count" : 1422
      }
    ]
  }
 }
}
```

## 参数说明

| 参数      | 必需/可选 | 数据类型  | 描述                                                                 |
| --------- | --------- | --------- | -------------------------------------------------------------------- |
| `field`   | 必填      | String    | 用于分桶的数值字段名称。                                             |
| `ranges`  | 必填      | Array     | 范围数组，每个元素可包含 `from`（含）和 `to`（不含）。               |
| `keyed`   | 可选      | Boolean   | 若为 `true`，以键值对形式返回桶。默认 `false`。                       |
| `script`  | 可选      | Object    | 用脚本生成分桶的值，替代 `field`。                                   |

> **注意**：每个范围的 `from` 值是 **包含** 的，`to` 值是 **不包含** 的（即 `[from, to)`）。

### 自定义分桶键名

默认的桶 key 使用 `"from-to"` 格式（如 `"1000.0-2000.0"`）。你可以为每个范围指定自定义的 key 使结果更易读：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "traffic_levels": {
      "range": {
        "field": "bytes",
        "keyed": true,
        "ranges": [
          { "key": "小流量", "to": 1000 },
          { "key": "中流量", "from": 1000, "to": 5000 },
          { "key": "大流量", "from": 5000 }
        ]
      }
    }
  }
}
```

### 嵌套子聚合

可以在每个范围桶内嵌套其他聚合，例如查看每个流量区间的平均响应码：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "bytes_ranges": {
      "range": {
        "field": "bytes",
        "ranges": [
          { "to": 1000 },
          { "from": 1000, "to": 5000 },
          { "from": 5000 }
        ]
      },
      "aggs": {
        "common_response": {
          "terms": {
            "field": "response.keyword",
            "size": 3
          }
        }
      }
    }
  }
}
```

> **提示**：如需对日期字段进行范围分桶，请使用 [日期范围聚合]({{< relref "./date-range.md" >}})；对 IP 地址使用 [IP 范围聚合]({{< relref "./ip-range.md" >}})。


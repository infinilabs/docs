---
title: "日期范围聚合（Date Range）"
date: 0001-01-01
summary: "日期范围聚合 #  date_range 聚合在概念上与 range 聚合相同，只是它允许执行日期计算。例如，你可以获取过去 10 天内的所有文档。为了使日期更易读，可以使用 format 参数包含格式：
相关指南（先读这些） #    聚合基础  聚合场景实践  时间序列建模  GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;number_of_bytes&#34;: { &#34;date_range&#34;: { &#34;field&#34;: &#34;@timestamp&#34;, &#34;format&#34;: &#34;MM-yyyy&#34;, &#34;ranges&#34;: [ { &#34;from&#34;: &#34;now-10d/d&#34;, &#34;to&#34;: &#34;now&#34; } ] } } } } 返回内容
... &#34;aggregations&#34; : { &#34;number_of_bytes&#34; : { &#34;buckets&#34; : [ { &#34;key&#34; : &#34;03-2021-03-2021&#34;, &#34;from&#34; : 1.6145568E12, &#34;from_as_string&#34; : &#34;03-2021&#34;, &#34;to&#34; : 1."
---


# 日期范围聚合

`date_range` 聚合在概念上与 `range` 聚合相同，只是它允许执行日期计算。例如，你可以获取过去 10 天内的所有文档。为了使日期更易读，可以使用 `format` 参数包含格式：

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})
- [时间序列建模]({{< relref "/docs/best-practices/data-modeling/time-series.md" >}})

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes": {
      "date_range": {
        "field": "@timestamp",
        "format": "MM-yyyy",
        "ranges": [
          {
            "from": "now-10d/d",
            "to": "now"
          }
        ]
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
        "key" : "03-2021-03-2021",
        "from" : 1.6145568E12,
        "from_as_string" : "03-2021",
        "to" : 1.615451329043E12,
        "to_as_string" : "03-2021",
        "doc_count" : 0
      }
    ]
  }
 }
}
```


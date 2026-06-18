---
title: "缺失值聚合（Missing）"
date: 0001-01-01
summary: "缺失值聚合 #  如果你的索引中的文档完全不包含聚合字段，或者聚合字段的值为 null，请使用 missing 参数指定这些文档应该放入的分组的名称。
相关指南（先读这些） #    聚合基础  聚合场景实践  以下示例将任何缺失的值添加到名为“N/A”的分组中：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;response_codes&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;response.keyword&#34;, &#34;size&#34;: 10, &#34;missing&#34;: &#34;N/A&#34; } } } } 由于 min_doc_count 参数的默认值为 1， missing 参数在其响应中不会返回任何分组。将 min_doc_count 参数设置为 0 以在响应中查看“N/A”分组：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;response_codes&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;response.keyword&#34;, &#34;size&#34;: 10, &#34;missing&#34;: &#34;N/A&#34;, &#34;min_doc_count&#34;: 0 } } } } 返回内容"
---


# 缺失值聚合

如果你的索引中的文档完全不包含聚合字段，或者聚合字段的值为 `null`，请使用 `missing` 参数指定这些文档应该放入的分组的名称。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

以下示例将任何缺失的值添加到名为“N/A”的分组中：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "size": 10,
        "missing": "N/A"
      }
    }
  }
}

```

由于 `min_doc_count` 参数的默认值为 1， `missing` 参数在其响应中不会返回任何分组。将 `min_doc_count` 参数设置为 0 以在响应中查看“N/A”分组：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "size": 10,
        "missing": "N/A",
        "min_doc_count": 0
      }
    }
  }
}
```

返回内容

```
...
"aggregations" : {
  "response_codes" : {
    "doc_count_error_upper_bound" : 0,
    "sum_other_doc_count" : 0,
    "buckets" : [
      {
        "key" : "200",
        "doc_count" : 12832
      },
      {
        "key" : "404",
        "doc_count" : 801
      },
      {
        "key" : "503",
        "doc_count" : 441
      },
      {
        "key" : "N/A",
        "doc_count" : 0
      }
    ]
  }
 }
}
```


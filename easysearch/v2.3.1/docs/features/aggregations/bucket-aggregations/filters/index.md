---
title: "多过滤器聚合（Filters）"
date: 0001-01-01
summary: "多过滤器聚合 #  filters 聚合与 filter 聚合相同，但它允许你使用多个过滤器聚合。filter 聚合结果为一个分组，而 filters 聚合会返回多个分组，每个定义的过滤器对应一个分组。
相关指南（先读这些） #    聚合基础  聚合场景实践  要为所有未匹配任何过滤器查询的文档创建一个分组，将 other_bucket 属性设置为 true ：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;200_os&#34;: { &#34;filters&#34;: { &#34;other_bucket&#34;: true, &#34;filters&#34;: [ { &#34;term&#34;: { &#34;response.keyword&#34;: &#34;200&#34; } }, { &#34;term&#34;: { &#34;machine.os.keyword&#34;: &#34;osx&#34; } } ] }, &#34;aggs&#34;: { &#34;avg_amount&#34;: { &#34;avg&#34;: { &#34;field&#34;: &#34;bytes&#34; } } } } } } 返回内容"
---


# 多过滤器聚合

`filters` 聚合与 `filter` 聚合相同，但它允许你使用多个过滤器聚合。`filter` 聚合结果为一个分组，而 `filters` 聚合会返回多个分组，每个定义的过滤器对应一个分组。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

要为所有未匹配任何过滤器查询的文档创建一个分组，将 `other_bucket` 属性设置为 `true` ：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "200_os": {
      "filters": {
        "other_bucket": true,
        "filters": [
          {
            "term": {
              "response.keyword": "200"
            }
          },
          {
            "term": {
              "machine.os.keyword": "osx"
            }
          }
        ]
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "bytes"
          }
        }
      }
    }
  }
}

```

返回内容

```
...
"aggregations" : {
  "200_os" : {
    "buckets" : [
      {
        "doc_count" : 12832,
        "avg_amount" : {
          "value" : 5897.852711970075
        }
      },
      {
        "doc_count" : 2825,
        "avg_amount" : {
          "value" : 5620.347256637168
        }
      },
      {
        "doc_count" : 1017,
        "avg_amount" : {
          "value" : 3247.0963618485744
        }
      }
    ]
  }
 }
}
```


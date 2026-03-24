---
title: "反向嵌套聚合（Reverse Nested）"
date: 0001-01-01
summary: "反向嵌套聚合 #  您可以将嵌套文档中的值聚合到其父文档中；这种聚合称为 reverse_nested。您可以使用 reverse_nested 在按嵌套对象中的字段分组后，聚合父文档中的字段。reverse_nested 聚合将&quot;连接回&quot;根页面，并为您的各种变体获取 load_time。
reverse_nested 聚合是嵌套聚合中的一个子聚合。它接受一个名为 path 的选项。此选项定义 Easysearch 在计算聚合时在文档层次结构中向后退多少步。
相关指南（先读这些） #    聚合基础  Nested 建模  聚合场景实践  GET logs/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;response&#34;: &#34;200&#34; } }, &#34;aggs&#34;: { &#34;pages&#34;: { &#34;nested&#34;: { &#34;path&#34;: &#34;pages&#34; }, &#34;aggs&#34;: { &#34;top_pages_per_load_time&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;pages.load_time&#34; }, &#34;aggs&#34;: { &#34;comment_to_logs&#34;: { &#34;reverse_nested&#34;: {}, &#34;aggs&#34;: { &#34;min_load_time&#34;: { &#34;min&#34;: { &#34;field&#34;: &#34;pages.load_time&#34; } } } } } } } } } } 返回内容"
---


# 反向嵌套聚合

您可以将嵌套文档中的值聚合到其父文档中；这种聚合称为 `reverse_nested`。您可以使用 `reverse_nested` 在按嵌套对象中的字段分组后，聚合父文档中的字段。`reverse_nested` 聚合将"连接回"根页面，并为您的各种变体获取 `load_time`。

`reverse_nested` 聚合是嵌套聚合中的一个子聚合。它接受一个名为 `path` 的选项。此选项定义 Easysearch 在计算聚合时在文档层次结构中向后退多少步。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [Nested 建模]({{< relref "/docs/best-practices/data-modeling/nested.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

```
GET logs/_search
{
  "query": {
    "match": { "response": "200" }
  },
  "aggs": {
    "pages": {
      "nested": {
        "path": "pages"
      },
      "aggs": {
        "top_pages_per_load_time": {
          "terms": {
            "field": "pages.load_time"
          },
          "aggs": {
            "comment_to_logs": {
              "reverse_nested": {},
              "aggs": {
                "min_load_time": {
                  "min": {
                    "field": "pages.load_time"
                  }
                }
              }
            }
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
  "pages" : {
    "doc_count" : 2,
    "top_pages_per_load_time" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 200.0,
          "doc_count" : 1,
          "comment_to_logs" : {
            "doc_count" : 1,
            "min_load_time" : {
              "value" : null
            }
          }
        },
        {
          "key" : 500.0,
          "doc_count" : 1,
          "comment_to_logs" : {
            "doc_count" : 1,
            "min_load_time" : {
              "value" : null
            }
          }
        }
      ]
    }
  }
 }
}
```

返回内容显示日志索引有一页带有 `load_time` 200，还有一页带有 `load_time` 500。


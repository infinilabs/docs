---
title: "邻接矩阵聚合（Adjacency Matrix）"
date: 0001-01-01
summary: "邻接矩阵聚合 #  adjacency_matrix 聚合允许你定义过滤表达式，并返回一个交集矩阵，矩阵中的每个非空单元格代表一个分组。你可以找到落入任何过滤器组合中的文档数量。
使用 adjacency_matrix 聚合通过将数据可视化为图形来发现概念之间的关联。
相关指南（先读这些） #    聚合基础  聚合场景实践  例如，下面查询可以分析不同制造公司之间的关联关系：
GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;interactions&#34;: { &#34;adjacency_matrix&#34;: { &#34;filters&#34;: { &#34;grpA&#34;: { &#34;match&#34;: { &#34;manufacturer.keyword&#34;: &#34;Low Tide Media&#34; } }, &#34;grpB&#34;: { &#34;match&#34;: { &#34;manufacturer.keyword&#34;: &#34;Elitelligence&#34; } }, &#34;grpC&#34;: { &#34;match&#34;: { &#34;manufacturer.keyword&#34;: &#34;Oceanavigations&#34; } } } } } } } 返回内容
{ ... &#34;aggregations&#34; : { &#34;interactions&#34; : { &#34;buckets&#34; : [ { &#34;key&#34; : &#34;grpA&#34;, &#34;doc_count&#34; : 1553 }, { &#34;key&#34; : &#34;grpA&amp;grpB&#34;, &#34;doc_count&#34; : 590 }, { &#34;key&#34; : &#34;grpA&amp;grpC&#34;, &#34;doc_count&#34; : 329 }, { &#34;key&#34; : &#34;grpB&#34;, &#34;doc_count&#34; : 1370 }, { &#34;key&#34; : &#34;grpB&amp;grpC&#34;, &#34;doc_count&#34; : 299 }, { &#34;key&#34; : &#34;grpC&#34;, &#34;doc_count&#34; : 1218 } ] } } } 让我们更仔细地查看结果"
---


# 邻接矩阵聚合

`adjacency_matrix` 聚合允许你定义过滤表达式，并返回一个交集矩阵，矩阵中的每个非空单元格代表一个分组。你可以找到落入任何过滤器组合中的文档数量。

使用 `adjacency_matrix` 聚合通过将数据可视化为图形来发现概念之间的关联。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

例如，下面查询可以分析不同制造公司之间的关联关系：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "interactions": {
      "adjacency_matrix": {
        "filters": {
          "grpA": {
            "match": {
              "manufacturer.keyword": "Low Tide Media"
            }
          },
          "grpB": {
            "match": {
              "manufacturer.keyword": "Elitelligence"
            }
          },
          "grpC": {
            "match": {
              "manufacturer.keyword": "Oceanavigations"
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
 {
   ...
   "aggregations" : {
     "interactions" : {
       "buckets" : [
         {
           "key" : "grpA",
           "doc_count" : 1553
         },
         {
           "key" : "grpA&grpB",
           "doc_count" : 590
         },
         {
           "key" : "grpA&grpC",
           "doc_count" : 329
         },
         {
           "key" : "grpB",
           "doc_count" : 1370
         },
         {
           "key" : "grpB&grpC",
           "doc_count" : 299
         },
         {
           "key" : "grpC",
           "doc_count" : 1218
         }
       ]
     }
   }
 }
```

让我们更仔细地查看结果

```
 {
    "key" : "grpA&grpB",
    "doc_count" : 590
 }
```

- `grpA` ：由 Low Tide Media 制造的产品。
- `grpB` ：由 Elitelligence 制造的产品。
- `590` : 同时生产的产品数量。


---
title: "采样聚合（Sampler）"
date: 0001-01-01
summary: "采样聚合 #  如果你正在聚合大量文档，可以使用 sampler 聚合将范围缩小到一小部分文档，从而获得更快的响应。sampler 聚合通过选择得分最高的文档来选取样本。
结果是大致的，但能很好地反映真实数据的分布。sampler 聚合显著提高了查询性能，但估计的响应并不完全可靠。
相关指南（先读这些） #    聚合基础  聚合性能优化  聚合场景实践  基本语法是：
“aggs”: { &#34;SAMPLE&#34;: { &#34;sampler&#34;: { &#34;shard_size&#34;: 100 }, &#34;aggs&#34;: {...} } } 分片大小属性 #  shard_size 属性告诉 Easysearch 每个分片最多收集多少文档。
以下示例将每个分片上收集的文档数量限制为 1,000，然后使用 terms 聚合对文档进行分组：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;sample&#34;: { &#34;sampler&#34;: { &#34;shard_size&#34;: 1000 }, &#34;aggs&#34;: { &#34;terms&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;agent.keyword&#34; } } } } } } 返回内容"
---


# 采样聚合

如果你正在聚合大量文档，可以使用 `sampler` 聚合将范围缩小到一小部分文档，从而获得更快的响应。`sampler` 聚合通过选择得分最高的文档来选取样本。

结果是大致的，但能很好地反映真实数据的分布。`sampler` 聚合显著提高了查询性能，但估计的响应并不完全可靠。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合性能优化]({{< relref "/docs/features/aggregations/aggs-performance.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

基本语法是：

```
“aggs”: {
  "SAMPLE": {
    "sampler": {
      "shard_size": 100
    },
    "aggs": {...}
  }
}
```

## 分片大小属性

`shard_size` 属性告诉 Easysearch 每个分片最多收集多少文档。

以下示例将每个分片上收集的文档数量限制为 1,000，然后使用 `terms` 聚合对文档进行分组：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sample": {
      "sampler": {
        "shard_size": 1000
      },
      "aggs": {
        "terms": {
          "terms": {
            "field": "agent.keyword"
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
  "sample" : {
    "doc_count" : 1000,
    "terms" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
          "doc_count" : 368
        },
        {
          "key" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
          "doc_count" : 329
        },
        {
          "key" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
          "doc_count" : 303
        }
      ]
    }
  }
 }
}
```


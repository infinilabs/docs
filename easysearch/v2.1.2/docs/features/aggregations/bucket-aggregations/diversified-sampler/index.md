---
title: "多样性采样聚合（Diversified Sampler）"
date: 0001-01-01
summary: "多样性采样聚合 #  diversified_sampler 聚合允许你通过去重包含相同 field 的文档来减少样本池分布的偏差。它通过使用 max_docs_per_value 和 field 设置来实现，这些设置限制了在分片上收集的 field 的最大文档数。max_docs_per_value 设置是一个可选参数，用于确定每个 field 将返回的最大文档数。此设置的默认值为 1。
与 sampler 聚合类似，你可以使用 shard_size 设置来控制在任何单个分片上收集的最大文档数，如下面的示例所示：
相关指南（先读这些） #    聚合基础  聚合性能优化  聚合场景实践  GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;sample&#34;: { &#34;diversified_sampler&#34;: { &#34;shard_size&#34;: 1000, &#34;field&#34;: &#34;response.keyword&#34; }, &#34;aggs&#34;: { &#34;terms&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;agent.keyword&#34; } } } } } } 返回内容
... &#34;aggregations&#34; : { &#34;sample&#34; : { &#34;doc_count&#34; : 3, &#34;terms&#34; : { &#34;doc_count_error_upper_bound&#34; : 0, &#34;sum_other_doc_count&#34; : 0, &#34;buckets&#34; : [ { &#34;key&#34; : &#34;Mozilla/5."
---


# 多样性采样聚合

`diversified_sampler` 聚合允许你通过去重包含相同 `field` 的文档来减少样本池分布的偏差。它通过使用 `max_docs_per_value` 和 `field` 设置来实现，这些设置限制了在分片上收集的 `field` 的最大文档数。`max_docs_per_value` 设置是一个可选参数，用于确定每个 `field` 将返回的最大文档数。此设置的默认值为 1。

与 `sampler` 聚合类似，你可以使用 `shard_size` 设置来控制在任何单个分片上收集的最大文档数，如下面的示例所示：

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合性能优化]({{< relref "/docs/features/aggregations/aggs-performance.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sample": {
      "diversified_sampler": {
        "shard_size": 1000,
        "field": "response.keyword"
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
    "doc_count" : 3,
    "terms" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
          "doc_count" : 2
        },
        {
          "key" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
          "doc_count" : 1
        }
      ]
    }
  }

 }
}
```
## 参数说明

| 参数                 | 必需/可选 | 数据类型 | 描述                                                                                   |
| -------------------- | --------- | -------- | -------------------------------------------------------------------------------------- |
| `field`              | 必填      | String   | 用于去重的字段。每个唯一值最多保留 `max_docs_per_value` 个文档。                         |
| `shard_size`         | 可选      | Integer  | 每个分片上收集的最大文档数。默认 100。                                                   |
| `max_docs_per_value` | 可选      | Integer  | 每个 `field` 值最多保留的文档数。默认 1。                                                |
| `execution_hint`     | 可选      | String   | 执行方式提示。可选 `map`、`global_ordinals`、`bytes_hash`。                               |

### sampler vs. diversified_sampler

| 聚合                  | 采样方式                                | 适用场景                            |
| --------------------- | --------------------------------------- | ----------------------------------- |
| `sampler`             | 取评分最高的前 N 个文档                 | 简单采样，快速获取近似结果          |
| `diversified_sampler`  | 按某字段去重后采样                     | 需要多样性，避免某类数据主导结果    |

> **提示**：`diversified_sampler` 适用于需要公平代表各个分类的场景，例如在多种类商品中均匀采样后再做聚合分析。

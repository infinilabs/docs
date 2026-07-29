---
title: "显著文本聚合（Significant Text）"
date: 0001-01-01
summary: "显著文本聚合 #  significant_text 聚合与 significant_terms 聚合类似，但它适用于原始文本字段。重要文本通过统计分析测量前景集和背景集之间流行度的变化。例如，当你搜索其股票缩写 TSLA 时，它可能会建议 Tesla。
significant_text 聚合会动态重新分析源文本，过滤掉重复段落、模板化的页眉和页脚等噪声数据，这些数据可能会扭曲结果。
重新分析高基数数据集可能是一项非常耗费 CPU 的操作。我们建议在采样聚合中使用 significant_text 聚合来将分析限制在少量最匹配文档中，例如 200。
相关指南（先读这些） #    聚合基础  聚合性能优化  聚合场景实践  您可以设置以下参数：
 size - 返回的最大桶数量。默认 10。 min_doc_count - 返回匹配超过配置数量的 Top Hits 结果。我们不建议将 min_doc_count 设置为 1，因为它倾向于返回拼写错误或错别字。找到一个以上的词项实例有助于加强显著性不是偶然事件的结果。默认值 3 用于提供最小证据权重。 shard_size - 设置高值会增加稳定性（和准确性），但会牺牲计算性能。 shard_min_doc_count - 如果你的文本包含许多低频词，而你又不关心这些词（例如拼写错误），那么你可以将 shard_min_doc_count 参数设置为在分片级别上过滤候选词，以合理地确保即使合并本地显著文本频率也不会达到所需的 min_doc_count 。默认值为 1，直到你显式设置它之前没有影响。我们建议将此值设置得远低于 min_doc_count 值。 filter_duplicate_text - 是否过滤掉重复的文本段落（如模板化页眉/页脚）。默认 true。设为 false 可提升性能，但可能引入噪声。  假设你在一个 Easysearch 集群中索引了莎士比亚的全部作品。你可以在 text_entry 字段中找到与“breathe”相关的显著文本："
---


# 显著文本聚合

`significant_text` 聚合与 `significant_terms` 聚合类似，但它适用于原始文本字段。重要文本通过统计分析测量前景集和背景集之间流行度的变化。例如，当你搜索其股票缩写 TSLA 时，它可能会建议 Tesla。

`significant_text` 聚合会动态重新分析源文本，过滤掉重复段落、模板化的页眉和页脚等噪声数据，这些数据可能会扭曲结果。

重新分析高基数数据集可能是一项非常耗费 CPU 的操作。我们建议在采样聚合中使用 `significant_text` 聚合来将分析限制在少量最匹配文档中，例如 200。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合性能优化]({{< relref "/docs/features/aggregations/aggs-performance.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

您可以设置以下参数：

- `size` - 返回的最大桶数量。默认 10。
- `min_doc_count` - 返回匹配超过配置数量的 Top Hits 结果。我们不建议将 `min_doc_count` 设置为 1，因为它倾向于返回拼写错误或错别字。找到一个以上的词项实例有助于加强显著性不是偶然事件的结果。默认值 3 用于提供最小证据权重。
- `shard_size` - 设置高值会增加稳定性（和准确性），但会牺牲计算性能。
- `shard_min_doc_count` - 如果你的文本包含许多低频词，而你又不关心这些词（例如拼写错误），那么你可以将 `shard_min_doc_count` 参数设置为在分片级别上过滤候选词，以合理地确保即使合并本地显著文本频率也不会达到所需的 `min_doc_count` 。默认值为 1，直到你显式设置它之前没有影响。我们建议将此值设置得远低于 `min_doc_count` 值。
- `filter_duplicate_text` - 是否过滤掉重复的文本段落（如模板化页眉/页脚）。默认 `true`。设为 `false` 可提升性能，但可能引入噪声。

假设你在一个 Easysearch 集群中索引了莎士比亚的全部作品。你可以在 `text_entry` 字段中找到与“breathe”相关的显著文本：

```
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "breathe"
    }
  },
  "aggregations": {
    "my_sample": {
      "sampler": {
        "shard_size": 100
      },
      "aggregations": {
        "keywords": {
          "significant_text": {
            "field": "text_entry",
            "min_doc_count": 4
          }
        }
      }
    }
  }
}

```

返回内容

```
"aggregations" : {
  "my_sample" : {
    "doc_count" : 59,
    "keywords" : {
      "doc_count" : 59,
      "bg_count" : 111396,
      "buckets" : [
        {
          "key" : "breathe",
          "doc_count" : 59,
          "score" : 1887.0677966101694,
          "bg_count" : 59
        },
        {
          "key" : "air",
          "doc_count" : 4,
          "score" : 2.641295376716233,
          "bg_count" : 189
        },
        {
          "key" : "dead",
          "doc_count" : 4,
          "score" : 0.9665839666414213,
          "bg_count" : 495
        },
        {
          "key" : "life",
          "doc_count" : 5,
          "score" : 0.9090787433467572,
          "bg_count" : 805
        }
      ]
    }
  }
 }
}
```

与 `breathe` 最相关的文本是 `air` 、 `dead` 和 `life` 。

`significant_text` 聚合有以下限制：

- 不支持子聚合，因为子聚合会带来较高的内存成本。作为解决方案，您可以使用带包含子句和子聚合的 terms 聚合添加一个后续查询。
- 不支持嵌套对象，因为它基于文档的 JSON 源进行工作。
- 文档计数可能会有一些（通常很小）的不准确，因为它基于从每个分片返回的样本进行求和。您可以使用 shard_size 参数来微调准确性和性能之间的权衡。默认情况下， shard_size 设置为 -1 以自动估计分片和 size 参数的数量。

统计信息中背景词频的默认来源是整个索引。您可以使用背景过滤器缩小此范围，以便更聚焦：

```
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "breathe"
    }
  },
  "aggregations": {
    "my_sample": {
      "sampler": {
        "shard_size": 100
      },
      "aggregations": {
        "keywords": {
          "significant_text": {
            "field": "text_entry",
            "background_filter": {
              "term": {
                "speaker": "JOHN OF GAUNT"
              }
            }
          }
        }
      }
    }
  }
}
```


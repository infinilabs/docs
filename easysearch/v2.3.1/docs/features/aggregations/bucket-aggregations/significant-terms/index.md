---
title: "显著词项聚合（Significant Terms）"
date: 0001-01-01
summary: "显著词项聚合 #  significant_terms 聚合可以帮助你在相对于索引中其他数据的过滤子集中识别不寻常或有趣的分组出现情况。
前景集是指你进行过滤的文档集合，背景集是指索引中所有文档的集合。significant_terms 聚合会检查前景集中的所有文档，并与背景集中的文档进行对比，从而为重要出现情况找到相应的分数。
相关指南（先读这些） #    聚合基础  聚合场景实践  在示例网络日志数据中，每个文档都有一个包含访客 user-agent 的字段。此示例搜索来自 iOS 操作系统的所有请求。对这一前景集进行常规的 terms 聚合返回 Firefox，因为它在这个分组内有最多的文档数量。另一方面， significant_terms 聚合返回 Internet Explorer（IE），因为 IE 在前景集中的出现频率显著高于背景集。
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;query&#34;: { &#34;terms&#34;: { &#34;machine.os.keyword&#34;: [ &#34;ios&#34; ] } }, &#34;aggs&#34;: { &#34;significant_response_codes&#34;: { &#34;significant_terms&#34;: { &#34;field&#34;: &#34;agent.keyword&#34; } } } } 返回内容
... &#34;aggregations&#34; : { &#34;significant_response_codes&#34; : { &#34;doc_count&#34; : 2737, &#34;bg_count&#34; : 14074, &#34;buckets&#34; : [ { &#34;key&#34; : &#34;Mozilla/4."
---


# 显著词项聚合

`significant_terms` 聚合可以帮助你在相对于索引中其他数据的过滤子集中识别不寻常或有趣的分组出现情况。

前景集是指你进行过滤的文档集合，背景集是指索引中所有文档的集合。`significant_terms` 聚合会检查前景集中的所有文档，并与背景集中的文档进行对比，从而为重要出现情况找到相应的分数。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

在示例网络日志数据中，每个文档都有一个包含访客 `user-agent` 的字段。此示例搜索来自 iOS 操作系统的所有请求。对这一前景集进行常规的 `terms` 聚合返回 Firefox，因为它在这个分组内有最多的文档数量。另一方面， `significant_terms` 聚合返回 Internet Explorer（IE），因为 IE 在前景集中的出现频率显著高于背景集。

```
GET sample_data_logs/_search
{
  "size": 0,
  "query": {
    "terms": {
      "machine.os.keyword": [
        "ios"
      ]
    }
  },
  "aggs": {
    "significant_response_codes": {
      "significant_terms": {
        "field": "agent.keyword"
      }
    }
  }
}

```

返回内容

```
...
"aggregations" : {
  "significant_response_codes" : {
    "doc_count" : 2737,
    "bg_count" : 14074,
    "buckets" : [
      {
        "key" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
        "doc_count" : 818,
        "score" : 0.01462731514608217,
        "bg_count" : 4010
      },
      {
        "key" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
        "doc_count" : 1067,
        "score" : 0.009062566630410223,
        "bg_count" : 5362
      }
    ]
  }
 }
}
```

如果 `significant_terms` 聚合没有返回任何结果，你可能没有使用查询来过滤结果。或者，前景集中词条的分布可能与背景集相同，这意味着前景集中没有什么异常。

背景词条频率统计信息的默认来源是整个索引。你可以使用背景过滤器来缩小这个范围，以便更聚焦

## 参数说明

| 参数                         | 必需/可选 | 数据类型 | 描述                                                                   |
| ---------------------------- | --------- | -------- | ---------------------------------------------------------------------- |
| `field`                      | 必填      | String   | 要分析的 keyword 或 text 字段。                                         |
| `size`                       | 可选      | Integer  | 返回的桶数量。默认 10。                                                 |
| `min_doc_count`              | 可选      | Integer  | 前景集中至少出现此次数的词项才纳入分析。默认 3。                         |
| `shard_min_doc_count`        | 可选      | Integer  | 分片级别的最小文档数。默认 1。                                           |
| `background_filter`          | 可选      | Object   | 自定义背景集的过滤条件，缩小对比范围。                                   |
| `mutual_information`         | 可选      | Object   | 使用互信息作为显著性评分算法。                                           |
| `chi_square`                 | 可选      | Object   | 使用卡方检验作为显著性评分算法。                                         |
| `gnd`                        | 可选      | Object   | 使用 Google 标准化距离作为评分算法。                                     |
| `jlh`                        | 可选      | Object   | 使用 JLH 评分（默认算法）。                                             |

### 实际应用：异常检测

`significant_terms` 是发现异常和关联的强大工具。典型场景：

- **安全分析**：在出现异常流量的 IP 段中，哪些 URL 路径显著高于正常水平？
- **电商推荐**：购买了某商品的用户，还显著倾向于购买哪些商品？
- **日志分析**：在报错的请求中，哪些参数组合异常频繁？

> **提示**：如果结果为空，通常意味着前景集的词频分布与背景集相似——即没有统计上的显著差异。尝试使用更具区分性的查询条件来定义前景集。


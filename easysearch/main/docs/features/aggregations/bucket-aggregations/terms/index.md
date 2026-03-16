---
title: "词项聚合（Terms）"
date: 0001-01-01
summary: "词项聚合 #  terms 聚合会动态为字段中的每个唯一词条创建一个分组。
相关指南（先读这些） #    聚合基础  聚合场景实践  以下示例使用 terms 聚合来查找网络日志数据中每个响应代码的文档数量：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;response_codes&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;response.keyword&#34;, &#34;size&#34;: 10 } } } } 返回内容
... &#34;aggregations&#34; : { &#34;response_codes&#34; : { &#34;doc_count_error_upper_bound&#34; : 0, &#34;sum_other_doc_count&#34; : 0, &#34;buckets&#34; : [ { &#34;key&#34; : &#34;200&#34;, &#34;doc_count&#34; : 12832 }, { &#34;key&#34; : &#34;404&#34;, &#34;doc_count&#34; : 801 }, { &#34;key&#34; : &#34;503&#34;, &#34;doc_count&#34; : 441 } ] } } } 值以 key 键返回。 doc_count 指定每个分组中的文档数量。默认情况下，分组按 doc-count 的降序排列。"
---


# 词项聚合

`terms` 聚合会动态为字段中的每个唯一词条创建一个分组。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

以下示例使用 `terms` 聚合来查找网络日志数据中每个响应代码的文档数量：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "size": 10
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
      }
    ]
  }
 }
}
```

值以 `key` 键返回。 `doc_count` 指定每个分组中的文档数量。默认情况下，分组按 `doc-count` 的降序排列。

> 可以使用 terms 通过按升序计数排序（ "order": {"count": "asc"} ）来搜索不常见的值。然而，我们强烈不建议这种做法，因为在涉及多个分片时，它可能导致不准确的结果。一个全局上不常见的词项可能不会在每个单个分片上显得不常见，或者可能完全不在某些分片返回的最不常见结果中。相反，一个在某个分片上出现频率较低的词项可能在另一个分片上很常见。在这两种情况下，分片级别的聚合可能会遗漏稀有词项，导致整体结果不正确。我们建议使用 rare_terms 聚合代替 terms 聚合，它专门设计用于更准确地处理这些情况。

## 返回文档数量和分片大小参数

`terms` 聚合返回的分组数量由 `size` 参数控制，默认值为 10。

此外，负责聚合的协调节点会提示每个分片返回其顶部唯一词项。每个分片返回的分组数量由 `shard_size` 参数控制。此参数与 `size` 参数不同，它作为增加分组文档计数准确性的机制存在。

例如，假设 `size` 和 `shard_size` 参数的值都为 3。 `terms` 聚合会向每个分片请求其前三个唯一词项。协调节点会汇总结果以计算最终结果。如果一个分片包含的某个对象不在前三个中，那么它就不会出现在响应中。然而，通过增加此请求的 `shard_size` 值，每个分片可以返回更多的唯一词项，从而提高协调节点收到所有相关结果的可能性。

默认情况下， `shard_size` 参数设置为 `size * 1.5 + 10` 。

在使用并发分片搜索时， `shard_size` 参数也会应用于每个分片切片。

`shard_size` 参数作为平衡 `terms` 聚合的性能和文档计数准确性的方式。较高的 `shard_size` 值将确保更高的文档计数准确性，但会导致更高的内存和计算使用。较低的 `shard_size` 值将更高效，但会导致较低的文档计数准确性。

## 文档计数错误

返回内容还包括两个名为 `doc_count_error_upper_bound` 和 `sum_other_doc_count` 的键。

`terms` 聚合返回最顶端的唯一词。因此，如果数据包含许多唯一词，那么其中一些可能不会出现在结果中。 `sum_other_doc_count` 字段表示被排除在响应之外的文档总和。在这种情况下，数字是 0，因为所有唯一值都出现在响应中。

`doc_count_error_upper_bound` 字段表示最终结果中排除的唯一值的最大可能计数。使用此字段来估计计数的误差范围。

`doc_count_error_upper_bound` 值和准确性的概念仅适用于使用默认排序方式（按文档数量降序）进行的聚合。这是因为当你按文档数量降序排序时，未返回的任何词项都保证包含等于或少于已返回词项的文档数。基于这一点，你可以计算 `doc_count_error_upper_bound` 。

## min_doc_count 和 shard_min_doc_count 参数

您可以使用 `min_doc_count` 参数过滤掉任何出现次数少于 `min_doc_count` 的唯一词项。 `min_doc_count` 阈值仅应用于合并从所有分片中检索到的结果之后。每个分片都不知道某个词项的全局文档数量。如果全局最频繁的 shard_size 个词项与某个分片本地最频繁的词项之间存在显著差异，在使用 `min_doc_count` 参数时您可能会收到意外结果。

另外， `shard_min_doc_count` 参数用于过滤掉分片返回给协调器且结果少于 `shard_min_doc_count` 的唯一词项。

## 收集模式

有两种收集模式可用： `depth_first` 和 `breadth_first` 。 `depth_first` 收集模式以深度优先方式扩展聚合树的所有分支，并在扩展完成后才进行剪枝。

然而，在使用嵌套的聚合时，返回的分组数量 `cardinality` 会被每个嵌套级别的字段 `cardinality` 相乘，这使得随着聚合的嵌套，分组数量容易出现组合爆炸。

你可以使用 `breadth_first` 集合模式来解决这个问题。在这种情况下，聚合树的第一级将在展开到下一级之前应用剪枝，可能会大大减少计算分组的数量。

此外，执行 `breadth_first` 收集还会带来内存开销，该开销与匹配文档的数量呈线性关系。这是因为 `breadth_first` 收集的工作方式是通过缓存并重放从父级层级修剪后的分组集。

## 参数说明

| 参数                          | 必需/可选 | 数据类型 | 描述                                                                                      |
| ----------------------------- | --------- | -------- | ----------------------------------------------------------------------------------------- |
| `field`                       | 必填      | String   | 用于分桶的字段名称。通常为 `keyword` 类型。                                                |
| `size`                        | 可选      | Integer  | 返回的桶数量。默认 10。                                                                    |
| `shard_size`                  | 可选      | Integer  | 每个分片返回的桶数量。默认 `size * 1.5 + 10`。值越大准确度越高，但性能开销也越大。           |
| `min_doc_count`               | 可选      | Integer  | 全局文档计数的最小阈值。低于此值的词项不返回。默认 1。                                      |
| `shard_min_doc_count`         | 可选      | Integer  | 分片级别的最小文档数过滤。默认无限制。                                                      |
| `order`                       | 可选      | Object   | 桶的排序方式。默认按 `_count` 降序。可按 `_key` 排序或按子聚合指标排序。                    |
| `missing`                     | 可选      | String   | 缺少字段值的文档所使用的替代值。默认忽略缺失值。                                            |
| `show_term_doc_count_error`   | 可选      | Boolean  | 在每个桶中显示 `doc_count_error_upper_bound`。默认 `false`。                                |
| `collect_mode`                | 可选      | String   | 聚合收集模式。可选 `depth_first`（默认）或 `breadth_first`。                                |
| `include`                     | 可选      | String/Array | 结果中要包含的词项。可以是正则表达式或值数组。                                           |
| `exclude`                     | 可选      | String/Array | 从结果中排除的词项。可以是正则表达式或值数组。                                           |
| `script`                      | 可选      | Object   | 使用脚本动态生成分桶值。                                                                   |

## 更改排序顺序

默认情况下，桶按 `doc_count` 降序排列。使用 `order` 参数可以改变排序方式：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}
```

也可以按子聚合的结果排序：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "order": {
          "avg_bytes": "desc"
        }
      },
      "aggs": {
        "avg_bytes": {
          "avg": {
            "field": "bytes"
          }
        }
      }
    }
  }
}
```

## 缺失值处理

默认情况下，没有聚合字段的文档会被忽略。使用 `missing` 参数为这些文档指定一个替代值：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "missing": "N/A"
      }
    }
  }
}
```

> **注意**：对于文本字段，`terms` 聚合会根据分析结果创建分组。分析过程（如词干化）会影响分组的精确性。建议在 `keyword` 类型的字段上使用 `terms` 聚合。

## 考虑预聚合数据

虽然 `doc_count` 字段提供了关于分组中聚合的独立文档数量的表示，但 `doc_count` 单独无法正确地增加存储预聚合数据的文档。要考虑预聚合数据并准确计算分组中的文档数量，您可以使用 `_doc_count` 字段来添加单个汇总字段中的文档数量。当文档包含 `_doc_count` 字段时，所有分组聚合都会识别其值并累积增加分组 `doc_count` 。在使用 `_doc_count` 字段时，请记住这些注意事项：

- 该字段不支持嵌套数组；只能使用正整数。
- 如果文档不包含 `_doc_count` 字段，聚合会使用该文档将计数增加 1。

## 参考样例

```
PUT /my_index/_doc/1
{
  "response_code": 404,
  "date":"2022-08-05",
  "_doc_count": 20
}

PUT /my_index/_doc/2
{
  "response_code": 404,
  "date":"2022-08-06",
  "_doc_count": 10
}

PUT /my_index/_doc/3
{
  "response_code": 200,
  "date":"2022-08-06",
  "_doc_count": 300
}

GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field" : "response_code"
      }
    }
  }
}
```

返回内容

```
{
  "took" : 20,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "response_codes" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 200,
          "doc_count" : 300
        },
        {
          "key" : 404,
          "doc_count" : 30
        }
      ]
    }
  }
}
```


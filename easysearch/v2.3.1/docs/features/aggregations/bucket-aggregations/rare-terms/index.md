---
title: "稀有词项聚合（Rare Terms）"
date: 0001-01-01
summary: "稀有词项聚合 #  rare_terms 聚合是一个分组聚合，用于识别数据集中的不常见词项。与 terms 聚合（查找最常见的词项）不同，rare_terms 聚合查找出现频率最低的词项。rare_terms 聚合适用于异常检测、长尾分析和异常报告等应用。
相关指南（先读这些） #    聚合基础  聚合场景实践   可以使用 terms 通过按升序计数排序（ &ldquo;order&rdquo;: {&ldquo;count&rdquo;: &ldquo;asc&rdquo;} ）来搜索不常见的值。然而，我们强烈不建议这种做法，因为在涉及多个分片时，它可能导致不准确的结果。一个全局上不常见的词项可能不会在每个单个分片上显得不常见，或者可能完全不在某些分片返回的最不常见结果中。相反，一个在某个分片上出现频率较低的词项可能在另一个分片上很常见。在这两种情况下，分片级别的聚合可能会遗漏稀有词项，导致整体结果不正确。我们建议使用 rare_terms 聚合代替 terms 聚合，它专门设计用于更准确地处理这些情况。
 近似结果 #  计算 rare_terms 聚合的精确结果需要编译所有分片上的值完整映射，这需要过多的运行时内存。因此， rare_terms 聚合结果被近似处理。
rare_terms 计算中的大多数错误是假阴性或“遗漏”的值，这些值定义了聚合检测测试的灵敏度。 rare_terms 聚合使用 CuckooFilter 算法以实现适当的灵敏度和可接受的内存使用平衡。有关 CuckooFilter 算法的描述，请参阅这篇论文。
控制灵敏度 #  rare_terms 聚合算法中的灵敏度误差被衡量为被遗漏的稀有值的比例，或 false negatives/target values 。例如，如果聚合在包含 5,000 个稀有值的数据集中遗漏了 100 个稀有值，灵敏度误差为 100/5000 = 0.02 ，或 2%。
您可以调整 precision 参数在 rare_terms 聚合中来控制灵敏度和内存使用之间的权衡。"
---


# 稀有词项聚合

`rare_terms` 聚合是一个分组聚合，用于识别数据集中的不常见词项。与 `terms` 聚合（查找最常见的词项）不同，`rare_terms` 聚合查找出现频率最低的词项。`rare_terms` 聚合适用于异常检测、长尾分析和异常报告等应用。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

> 可以使用 terms 通过按升序计数排序（ "order": {"count": "asc"} ）来搜索不常见的值。然而，我们强烈不建议这种做法，因为在涉及多个分片时，它可能导致不准确的结果。一个全局上不常见的词项可能不会在每个单个分片上显得不常见，或者可能完全不在某些分片返回的最不常见结果中。相反，一个在某个分片上出现频率较低的词项可能在另一个分片上很常见。在这两种情况下，分片级别的聚合可能会遗漏稀有词项，导致整体结果不正确。我们建议使用 rare_terms 聚合代替 terms 聚合，它专门设计用于更准确地处理这些情况。

## 近似结果

计算 `rare_terms` 聚合的精确结果需要编译所有分片上的值完整映射，这需要过多的运行时内存。因此， `rare_terms` 聚合结果被近似处理。

`rare_terms` 计算中的大多数错误是假阴性或“遗漏”的值，这些值定义了聚合检测测试的灵敏度。 `rare_terms` 聚合使用 CuckooFilter 算法以实现适当的灵敏度和可接受的内存使用平衡。有关 CuckooFilter 算法的描述，请参阅这篇论文。

## 控制灵敏度

`rare_terms` 聚合算法中的灵敏度误差被衡量为被遗漏的稀有值的比例，或 `false negatives/target values` 。例如，如果聚合在包含 5,000 个稀有值的数据集中遗漏了 100 个稀有值，灵敏度误差为 `100/5000 = 0.02` ，或 2%。

您可以调整 `precision` 参数在 `rare_terms` 聚合中来控制灵敏度和内存使用之间的权衡。

这些因素也会影响灵敏度和内存的权衡：

- 唯一值的总数
- 数据集中稀有项的比例

以下指南可以帮助你决定使用哪个 `precision` 值。

### 计算内存使用

运行时内存使用以绝对值描述，通常以 RAM 的 MB 为单位。

内存使用随唯一项数量的增加而线性增长。线性扩展系数因 `precision` 参数而异，大致在每百万个唯一值 1.0 到 2.5 MB 之间。对于默认的 `precision` 设置为 0.001 时，内存成本约为每百万个唯一值 1.75 MB。

### 管理灵敏度误差

敏感性误差随唯一值总数的增加而线性增长。有关估算唯一值数量的信息，请参阅基数聚合。

在默认 `precision` 设置下，即使对于具有 1000 万至 2000 万个唯一值的集合，敏感性误差也极少超过 2.5%。对于 `precision` 设置为 0.00001 时，敏感性误差极少超过 0.6%。然而，非常低的稀有值绝对数量可能导致误差率出现较大波动（如果有两个稀有值，漏掉其中一个会导致 50% 的误差率）。

### 与其他聚合的兼容性

`rare_terms` 聚合使用广度优先收集模式，与某些子聚合和嵌套配置中需要深度优先收集模式的聚合不兼容。

## 参数说明

`rare_terms` 聚合采用以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| ------------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------ |
| `field` | 必填 | String | 用于分析稀有词的字段。必须是数值类型或具有 `keyword` 映射的文本类型。|
| `max_doc_count` | 可选 | Integer | 一个词被考虑为罕见词所需的最大文档数量。默认值是 1 。最大值是 100 。|
| `precision` | 可选 | Integer | 控制用于识别罕见词的算法的精确度。较高的值提供更精确的结果，但会消耗更多内存。默认值是 0.001 。最小（最精确的允许值）是 0.00001 。|
| `include` | 可选 | Array/regex | 结果中要包含的词。可以是正则表达式或值的数组。|
| `exclude` | 可选 | Array/regex | 从结果中排除的词项。可以是正则表达式或值数组。|
| `missing` | 可选 | String | 用于没有聚合字段值的文档的值。|

## 参考样例

以下请求数据中仅出现一次的所有目的地机场代码：

```
GET sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "rare_destination": {
      "rare_terms": {
        "field": "DestAirportID",
        "max_doc_count": 1
      }
    }
  }
}

```

返回内容显示，有两个机场符合仅在数据中出现一次的标准：

```
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "rare_destination": {
      "buckets": [
        {
          "key": "ADL",
          "doc_count": 1
        },
        {
          "key": "BUF",
          "doc_count": 1
        }
      ]
    }
  }
}

```

### 文档数量限制

使用 `max_doc_count` 参数指定 `rare_terms` 聚合可以返回的最大文档数量。 `rare_terms` 返回的词项数量没有限制，因此较大的 `max_doc_count` 值可能会返回非常大的结果集。因此， 100 是允许的最大 `max_doc_count` 值。

以下请求数据中最多出现两次的所有目的地机场代码：

```
GET sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "rare_destination": {
      "rare_terms": {
        "field": "DestAirportID",
        "max_doc_count": 2
      }
    }
  }
}

```

响应显示，有七个目的地机场代码符合出现次数在两次或更少文档中的标准，包括前一个示例中的两个：

```
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "rare_destination": {
      "buckets": [
        {
          "key": "ADL",
          "doc_count": 1
        },
        {
          "key": "BUF",
          "doc_count": 1
        },
        {
          "key": "ABQ",
          "doc_count": 2
        },
        {
          "key": "AUH",
          "doc_count": 2
        },
        {
          "key": "BIL",
          "doc_count": 2
        },
        {
          "key": "BWI",
          "doc_count": 2
        },
        {
          "key": "MAD",
          "doc_count": 2
        }
      ]
    }
  }
}
```

## 过滤 (include 和 exclude)

使用 `include` 和 `exclude` 参数来过滤 `rare_terms` 聚合返回的值。这两个参数可以在同一个聚合中包含。 `exclude` 过滤器具有优先级；任何被排除的值都会从结果中移除，无论它们是否被明确包含。

`include` 和 `exclude` 的参数可以是正则表达式（`regex`），包括字符串字面量，或数组。混合正则表达式和数组参数会导致错误。例如，以下组合是不允许的：

```
"rare_terms": {
  "field": "DestAirportID",
  "max_doc_count": 2,
  "exclude": ["ABQ", "AUH"],
  "include": "A.*"
}
```

### 示例：过滤

以下示例修改了前面的示例，以包含所有以“A”开头的机场代码，但排除“ABQ”机场代码：

```
GET sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "rare_destination": {
      "rare_terms": {
        "field": "DestAirportID",
        "max_doc_count": 2,
        "include": "A.*",
        "exclude": "ABQ"
      }
    }
  }
}

```

响应显示了符合过滤条件的那两个机场代码：

```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "rare_destination": {
      "buckets": [
        {
          "key": "ADL",
          "doc_count": 1
        },
        {
          "key": "AUH",
          "doc_count": 2
        }
      ]
    }
  }
}
```

### 示例：使用数组输入进行过滤

以下示例数据中最多出现两次的 所有目的地机场代码，但指定了要排除的机场代码数组：

```
GET sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "rare_destination": {
      "rare_terms": {
        "field": "DestAirportID",
        "max_doc_count": 2,
        "exclude": ["ABQ", "BIL", "MAD"]
      }
    }
  }
}

```

响应中排除了排除的机场代码：

```
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "rare_destination": {
      "buckets": [
        {
          "key": "ADL",
          "doc_count": 1
        },
        {
          "key": "BUF",
          "doc_count": 1
        },
        {
          "key": "AUH",
          "doc_count": 2
        },
        {
          "key": "BWI",
          "doc_count": 2
        }
      ]
    }
  }
}

```


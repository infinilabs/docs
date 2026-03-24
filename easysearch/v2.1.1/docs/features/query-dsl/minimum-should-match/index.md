---
title: "Minimum Should Match 参数"
date: 0001-01-01
summary: "Minimum Should Match 参数 #  minimum_should_match 参数可用于全文搜索，并指定文档必须匹配的最小词项数量才能在搜索结果中返回。
以下示例要求文档至少匹配三个搜索词中的两个才能作为搜索结果返回：
相关指南（先读这些） #    查询 DSL 基础  全文搜索  GET /shakespeare/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;text_entry&#34;: { &#34;query&#34;: &#34;prince king star&#34;, &#34;minimum_should_match&#34;: &#34;2&#34; } } } } 在这个示例中，查询有三个可选子句，它们通过 OR 结合，因此文档必须匹配 prince 和 king ，或者 prince 和 star ，或者 king 和 star 。
参数值说明 #  您可以指定 minimum_should_match 参数为以下值之一。
   值类型 示例 描述     非负整数 2 一个文档必须匹配这个数量的可选子句。   负整数 -1 一个文档必须匹配可选子句总数减去这个数。   非负百分比 70% 一个文档必须匹配可选子句总数的这个百分比。要匹配的子句数向下取整到最接近的整数。   负百分比 -30% 一个文档可以有这个百分比的不匹配的可选子句。文档允许不匹配的子句数向下取整到最接近的整数。   组合 2&lt;75% n&lt;p% 格式的表达式。如果可选子句的数量小于或等于 n ，文档必须匹配所有可选子句。如果可选子句的数量大于 n ，则文档必须匹配 p 百分比的可选子句。   多种组合 3&lt;-1 5&lt;50% 用空格分隔的多个组合。每个条件适用于 &lt; 符号左侧数字更多的可选子句数量。在这个例子中，如果有三个或更少可选子句，文档必须匹配所有它们。如果有四或五个可选子句，文档必须匹配所有但一个。如果有 6 个或更多可选子句，文档必须匹配 50% 的它们。     设 n 为文档必须匹配的可选子句数量。当 n 计算为百分比时，如果 n 小于 1，则使用 1。如果 n 大于可选子句的数量，则使用可选子句的数量。"
---


# Minimum Should Match 参数

`minimum_should_match` 参数可用于全文搜索，并指定文档必须匹配的最小词项数量才能在搜索结果中返回。

以下示例要求文档至少匹配三个搜索词中的两个才能作为搜索结果返回：

## 相关指南（先读这些）

- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

```
GET /shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "prince king star",
        "minimum_should_match": "2"
      }
    }
  }
}
```

在这个示例中，查询有三个可选子句，它们通过 `OR` 结合，因此文档必须匹配 `prince` 和 `king` ，或者 `prince` 和 `star` ，或者 `king` 和 `star` 。

## 参数值说明

您可以指定 `minimum_should_match` 参数为以下值之一。

| 值类型     | 示例       | 描述                                                                                                                                                                                                                                      |
| ---------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 非负整数   | 2          | 一个文档必须匹配这个数量的可选子句。                                                                                                                                                                                                      |
| 负整数     | -1         | 一个文档必须匹配可选子句总数减去这个数。                                                                                                                                                                                                  |
| 非负百分比 | 70%        | 一个文档必须匹配可选子句总数的这个百分比。要匹配的子句数向下取整到最接近的整数。                                                                                                                                                          |
| 负百分比   | -30%       | 一个文档可以有这个百分比的不匹配的可选子句。文档允许不匹配的子句数向下取整到最接近的整数。                                                                                                                                                |
| 组合       | 2<75%      | n<p% 格式的表达式。如果可选子句的数量小于或等于 n ，文档必须匹配所有可选子句。如果可选子句的数量大于 n ，则文档必须匹配 p 百分比的可选子句。                                                                                              |
| 多种组合   | 3<-1 5<50% | 用空格分隔的多个组合。每个条件适用于 < 符号左侧数字更多的可选子句数量。在这个例子中，如果有三个或更少可选子句，文档必须匹配所有它们。如果有四或五个可选子句，文档必须匹配所有但一个。如果有 6 个或更多可选子句，文档必须匹配 50% 的它们。 |

> 设 n 为文档必须匹配的可选子句数量。当 n 计算为百分比时，如果 n 小于 1，则使用 1。如果 n 大于可选子句的数量，则使用可选子句的数量。

## 在布尔查询中使用参数

布尔查询在 `should` 子句中列出可选子句，在 `must` 子句中列出必需子句。可选地，它还可以包含一个 `filter` 子句来过滤结果。

考虑一个包含以下五个文档的示例索引：

```
PUT testindex/_doc/1
{
  "text": "one Easysearch"
}

PUT testindex/_doc/2
{
  "text": "one two Easysearch"
}

PUT testindex/_doc/3
{
  "text": "one two three Easysearch"
}

PUT testindex/_doc/4
{
  "text": "one two three four Easysearch"
}

PUT testindex/_doc/5
{
  "text": "Easysearch"
}
```

以下查询包含四个可选子句：

```
GET testindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text": "Easysearch"
          }
        }
      ],
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ],
      "minimum_should_match": "80%"
    }
  }
}
```

因为 `minimum_should_match` 被指定为 80% ，匹配可选子句的数量计算为 4 · 0.8 = 3.2，然后向下取整为 3。因此，结果包含至少匹配三个子句的文档：

```
{
  "took": 40,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 2.494999,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.494999,
        "_source": {
          "text": "one two three four Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "3",
        "_score": 1.5744598,
        "_source": {
          "text": "one two three Easysearch"
        }
      }
    ]
  }
}
```

现在将 `minimum_should_match` 指定为 `-20%` :

```
GET testindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text": "Easysearch"
          }
        }
      ],
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ],
      "minimum_should_match": "-20%"
    }
  }
}

```

一个文档中可以有的不匹配的可选子句的数量计算为 4 · 0.2 = 0.8，并向下取整为 0。因此，结果只包含一个匹配所有可选子句的文档：

```
{
  "took": 41,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 2.494999,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.494999,
        "_source": {
          "text": "one two three four Easysearch"
        }
      }
    ]
  }
}
```

请注意，指定一个正百分比（ 80% ）和一个负百分比（ -20% ）并没有导致文档必须匹配的可选子句数量相同，因为在这两种情况下，结果都被向下取整。如果可选子句的数量，例如为 5，那么 80% 和 -20% 都会产生相同数量的文档必须匹配的可选子句（4）。

### 默认 minimum_should_match 值

如果一个查询包含 `must` 或 `filter` 子句，默认 `minimum_should_match` 值为 0。例如，以下查询搜索匹配 `Easysearch` 和 0 个可选 `should` 子句的文档：

```
GET testindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text": "Easysearch"
          }
        }
      ],
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ]
    }
  }
}
```

这个查询返回索引中的所有五个文档：

```
{
  "took": 34,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 2.494999,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.494999,
        "_source": {
          "text": "one two three four Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "3",
        "_score": 1.5744598,
        "_source": {
          "text": "one two three Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.91368985,
        "_source": {
          "text": "one two Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.4338556,
        "_source": {
          "text": "one Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "5",
        "_score": 0.11964063,
        "_source": {
          "text": "Easysearch"
        }
      }
    ]
  }
}

```

然而，如果你省略了 `must` 子句，那么查询将搜索匹配一个可选的 `should` 子句的文档：

```
GET testindex/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ]
    }
  }
}

```

结果只包含四个至少匹配一个可选子句的文档：

```
{
  "took": 19,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 2.426633,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.426633,
        "_source": {
          "text": "one two three four Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "3",
        "_score": 1.4978898,
        "_source": {
          "text": "one two three Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.8266785,
        "_source": {
          "text": "one two Easysearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.3331056,
        "_source": {
          "text": "one Easysearch"
        }
      }
    ]
  }
}

```


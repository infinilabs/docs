---
title: "Has Child 查询"
date: 0001-01-01
summary: "Has Child 查询 #  has_child 查询返回匹配特定查询的子文档的父文档。您可以通过使用连接字段类型在相同索引中的文档之间建立父子关系。
相关指南（先读这些） #    Parent-Child 建模  关联查询（Joining）   性能注意：has_child 查询比其他查询慢，因为它执行了连接操作。随着指向不同父文档的匹配子文档数量的增加，性能会降低。您搜索中的每个 has_child 查询都可能显著影响查询性能。如果您优先考虑速度，请避免使用此查询或尽可能限制其使用。更多性能考虑，请参考 Parent-Child 建模章节。
 参考样例 #  在您运行一个 has_child 查询之前，您的索引必须包含一个连接字段，以便建立父子关系。索引映射请求使用以下格式：
PUT /example_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;relationship_field&#34;: { &#34;type&#34;: &#34;join&#34;, &#34;relations&#34;: { &#34;parent_doc&#34;: &#34;child_doc&#34; } } } } } 在这个例子中，您将配置一个包含代表产品和其品牌的文档的索引。
首先，创建索引并建立 brand 和 product 之间的父子关系：
PUT testindex1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;product_to_brand&#34;: { &#34;type&#34;: &#34;join&#34;, &#34;relations&#34;: { &#34;brand&#34;: &#34;product&#34; } } } } } 创建两个父（品牌）文档："
---


# Has Child 查询

`has_child` 查询返回匹配特定查询的子文档的父文档。您可以通过使用连接字段类型在相同索引中的文档之间建立父子关系。

## 相关指南（先读这些）

- [Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})
- [关联查询（Joining）]({{< relref "/docs/features/query-dsl/joining/_index.md" >}})

> **性能注意**：has_child 查询比其他查询慢，因为它执行了连接操作。随着指向不同父文档的匹配子文档数量的增加，性能会降低。您搜索中的每个 has_child 查询都可能显著影响查询性能。如果您优先考虑速度，请避免使用此查询或尽可能限制其使用。更多性能考虑，请参考[Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})章节。

## 参考样例

在您运行一个 `has_child` 查询之前，您的索引必须包含一个连接字段，以便建立父子关系。索引映射请求使用以下格式：

```
PUT /example_index
{
  "mappings": {
    "properties": {
      "relationship_field": {
        "type": "join",
        "relations": {
          "parent_doc": "child_doc"
        }
      }
    }
  }
}
```

在这个例子中，您将配置一个包含代表产品和其品牌的文档的索引。

首先，创建索引并建立 `brand` 和 `product` 之间的父子关系：

```
PUT testindex1
{
  "mappings": {
    "properties": {
      "product_to_brand": {
        "type": "join",
        "relations": {
          "brand": "product"
        }
      }
    }
  }
}
```

创建两个父（品牌）文档：

```
PUT testindex1/_doc/1
{
  "name": "Luxury brand",
  "product_to_brand" : "brand"
}

PUT testindex1/_doc/2
{
  "name": "Economy brand",
  "product_to_brand" : "brand"
}
```

索引三个子（产品）文档：

```
PUT testindex1/_doc/3?routing=1
{
  "name": "Mechanical watch",
  "sales_count": 150,
  "product_to_brand": {
    "name": "product",
    "parent": "1"
  }
}

PUT testindex1/_doc/4?routing=2
{
  "name": "Electronic watch",
  "sales_count": 300,
  "product_to_brand": {
    "name": "product",
    "parent": "2"
  }
}

PUT testindex1/_doc/5?routing=2
{
  "name": "Digital watch",
  "sales_count": 100,
  "product_to_brand": {
    "name": "product",
    "parent": "2"
  }
}
```

要搜索子文档的父文档，请使用一个 `has_child` 查询。以下查询返回制造手表的父文档（品牌）：

```
GET testindex1/_search
{
  "query" : {
    "has_child": {
      "type":"product",
      "query": {
        "match" : {
            "name": "watch"
        }
      }
    }
  }
}
```

返回了两个品牌：

```
{
  "took": 15,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Luxury brand",
          "product_to_brand": "brand"
        }
      },
      {
        "_index": "testindex1",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Economy brand",
          "product_to_brand": "brand"
        }
      }
    ]
  }
}
```

## 检索内部命中项

返回与查询匹配的子文档，请提供 `inner_hits` 参数：

```
GET testindex1/_search
{
  "query" : {
    "has_child": {
      "type":"product",
      "query": {
        "match" : {
            "name": "watch"
        }
      },
      "inner_hits": {}
    }
  }
}
```

包含子文档在 inner_hits 字段中：

```
{
  "took": 52,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Luxury brand",
          "product_to_brand": "brand"
        },
        "inner_hits": {
          "product": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 0.53899646,
              "hits": [
                {
                  "_index": "testindex1",
                  "_id": "3",
                  "_score": 0.53899646,
                  "_routing": "1",
                  "_source": {
                    "name": "Mechanical watch",
                    "sales_count": 150,
                    "product_to_brand": {
                      "name": "product",
                      "parent": "1"
                    }
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index": "testindex1",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Economy brand",
          "product_to_brand": "brand"
        },
        "inner_hits": {
          "product": {
            "hits": {
              "total": {
                "value": 2,
                "relation": "eq"
              },
              "max_score": 0.53899646,
              "hits": [
                {
                  "_index": "testindex1",
                  "_id": "4",
                  "_score": 0.53899646,
                  "_routing": "2",
                  "_source": {
                    "name": "Electronic watch",
                    "sales_count": 300,
                    "product_to_brand": {
                      "name": "product",
                      "parent": "2"
                    }
                  }
                },
                {
                  "_index": "testindex1",
                  "_id": "5",
                  "_score": 0.53899646,
                  "_routing": "2",
                  "_source": {
                    "name": "Digital watch",
                    "sales_count": 100,
                    "product_to_brand": {
                      "name": "product",
                      "parent": "2"
                    }
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

## 参数说明

以下表格列出了所有由 `has_child` 查询支持的一级参数。

| 参数              | 必填/可选 | 描述                                                                                                                                                                                                                                                                                                                                                     |
| ----------------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`            | 必填      | 指定在 `join` 字段映射中定义的子关系名称。                                                                                                                                                                                                                                                                                                               |
| `query`           | 必填      | 在子文档上运行的查询。如果子文档与查询匹配，则返回父文档。                                                                                                                                                                                                                                                                                               |
| `ignore_unmapped	` | 可选      | 指示是否忽略未映射的 type 字段，而不是抛出错误而不返回文档。您可以在查询多个索引时提供此参数，其中一些索引可能不包含 type 字段。默认为 false 。                                                                                                                                                                                                          |
| `max_children`    | 可选      | 父文档匹配子文档的最大数量。如果超过此数量，父文档将不会出现在搜索结果中。                                                                                                                                                                                                                                                                               |
| `min_children`    | 可选      | 父文档要包含在结果中所需的最小子文档匹配数量。如果未达到此要求，父文档将被排除。默认值为 1 。                                                                                                                                                                                                                                                            |
| `score_mode`      | 可选      | 定义匹配子文档的分数如何影响父文档的分数。有效值有：<br>- none : 忽略子文档的相关性分数，并将分数 0 分配给父文档。<br>- avg : 使用所有匹配子文档的平均相关性分数。<br>- max ：将匹配的子文档中的最高相关性得分分配给父文档。<br>- min ：将匹配的子文档中的最低相关性得分分配给父文档。<br>- sum ：将所有匹配的子文档的相关性得分相加。<br>默认为 none 。 |
| `inner_hits`      | 可选      | 如果提供，则返回与查询匹配的基础命中（子文档）。                                                                                                                                                                                                                                                                                                         |

## 排序限制

`has_child` 查询不支持使用标准排序选项对结果进行排序。如果您需要按子文档中的字段对父文档进行排序，可以使用 `function_score` 查询并按父文档的得分进行排序。

在前面示例中，您可以根据子产品（品牌）的 `sales_count` 对父文档进行排序。此查询将分数乘以子文档的 `sales_count` 字段，并将匹配的子文档中的最高相关性分数分配给父文档：

```
GET testindex1/_search
{
  "query": {
    "has_child": {
      "type": "product",
      "query": {
        "function_score": {
          "script_score": {
            "script": "_score * doc['sales_count'].value"
          }
        }
      },
      "score_mode": "max"
    }
  }
}
```

响应包含按最高子 `sales_count` 排序的品牌：

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 300,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "2",
        "_score": 300,
        "_source": {
          "name": "Economy brand",
          "product_to_brand": "brand"
        }
      },
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 150,
        "_source": {
          "name": "Luxury brand",
          "product_to_brand": "brand"
        }
      }
    ]
  }
}
```


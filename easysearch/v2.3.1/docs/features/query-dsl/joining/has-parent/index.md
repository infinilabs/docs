---
title: "Has Parent 查询"
date: 0001-01-01
summary: "Has Parent 查询 #  has_parent 查询返回匹配特定查询的父文档的子文档。您可以通过使用连接字段类型在相同索引中的文档之间建立父子关系。
相关指南（先读这些） #    Parent-Child 建模  关联查询（Joining）   性能注意：has_parent 查询比其他查询慢，因为它执行了连接操作。随着匹配的父文档数量的增加，性能会降低。您搜索中的每个 has_parent 查询都可能显著影响查询性能。如果您优先考虑速度，请避免使用此查询或尽可能限制其使用。更多性能考虑，请参考 Parent-Child 建模章节。
 参考样例 #  在您运行一个 has_parent 查询之前，您的索引必须包含一个连接字段，以便建立父子关系。索引映射请求使用以下格式：
PUT /example_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;relationship_field&#34;: { &#34;type&#34;: &#34;join&#34;, &#34;relations&#34;: { &#34;parent_doc&#34;: &#34;child_doc&#34; } } } } } 对于这个示例，首先配置一个包含代表产品和其品牌的文档的索引，这些文档如查询示例 has_child 中所述。
要搜索父项的子项，请使用 has_parent 查询。以下查询返回与查询 economy 匹配的品牌生产的子文档（产品）：
GET testindex1/_search { &#34;query&#34; : { &#34;has_parent&#34;: { &#34;parent_type&#34;:&#34;brand&#34;, &#34;query&#34;: { &#34;match&#34; : { &#34;name&#34;: &#34;economy&#34; } } } } } 返回由该品牌生产的所有产品："
---


# Has Parent 查询

`has_parent` 查询返回匹配特定查询的父文档的子文档。您可以通过使用连接字段类型在相同索引中的文档之间建立父子关系。

## 相关指南（先读这些）

- [Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})
- [关联查询（Joining）]({{< relref "/docs/features/query-dsl/joining/_index.md" >}})

> **性能注意**：has_parent 查询比其他查询慢，因为它执行了连接操作。随着匹配的父文档数量的增加，性能会降低。您搜索中的每个 has_parent 查询都可能显著影响查询性能。如果您优先考虑速度，请避免使用此查询或尽可能限制其使用。更多性能考虑，请参考[Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})章节。

## 参考样例

在您运行一个 `has_parent` 查询之前，您的索引必须包含一个连接字段，以便建立父子关系。索引映射请求使用以下格式：

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

对于这个示例，首先配置一个包含代表产品和其品牌的文档的索引，这些文档如查询示例 `has_child` 中所述。

要搜索父项的子项，请使用 `has_parent` 查询。以下查询返回与查询 `economy` 匹配的品牌生产的子文档（产品）：

```
GET testindex1/_search
{
  "query" : {
    "has_parent": {
      "parent_type":"brand",
      "query": {
        "match" : {
          "name": "economy"
        }
      }
    }
  }
}
```

返回由该品牌生产的所有产品：

```
{
  "took": 11,
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
        "_id": "4",
        "_score": 1,
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
        "_score": 1,
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
```

## 检索内部命中项

返回与查询匹配的父文档，请提供 `inner_hits` 参数：

```
GET testindex1/_search
{
  "query" : {
    "has_parent": {
      "parent_type":"brand",
      "query": {
        "match" : {
          "name": "economy"
        }
      },
      "inner_hits": {}
    }
  }
}
```

包含父文档在 `inner_hits` 字段中：

```
{
  "took": 11,
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
        "_id": "4",
        "_score": 1,
        "_routing": "2",
        "_source": {
          "name": "Electronic watch",
          "sales_count": 300,
          "product_to_brand": {
            "name": "product",
            "parent": "2"
          }
        },
        "inner_hits": {
          "brand": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.3862942,
              "hits": [
                {
                  "_index": "testindex1",
                  "_id": "2",
                  "_score": 1.3862942,
                  "_source": {
                    "name": "Economy brand",
                    "product_to_brand": "brand"
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index": "testindex1",
        "_id": "5",
        "_score": 1,
        "_routing": "2",
        "_source": {
          "name": "Digital watch",
          "sales_count": 100,
          "product_to_brand": {
            "name": "product",
            "parent": "2"
          }
        },
        "inner_hits": {
          "brand": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.3862942,
              "hits": [
                {
                  "_index": "testindex1",
                  "_id": "2",
                  "_score": 1.3862942,
                  "_source": {
                    "name": "Economy brand",
                    "product_to_brand": "brand"
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

以下表格列出了所有由 `has_parent` 查询支持的一级参数。

| 参数              | 必填/可选 | 描述                                                                                                                                                                                                                                       |
| ----------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `parent_type`     | 必填      | 指定在 `join` 字段映射中定义的父级关系的名称。                                                                                                                                                                                             |
| `query`           | 必填      | 在父级文档上运行的查询。如果父级文档与查询匹配，则返回子文档。                                                                                                                                                                             |
| `ignore_unmapped	` | 可选      | 指示是否忽略未映射的 `parent_type` 字段，而不是抛出错误而不返回文档。您可以在查询多个索引时提供此参数，其中一些索引可能不包含 `parent_type` 字段。默认为 false 。                                                                          |
| `score`           | 可选      | Boolean。是否将匹配的父文档的相关性分数汇总到子文档中。`false`（默认）——忽略父文档分数，子文档分数等于 `boost`（默认 1）。`true`——父文档的相关性分数会汇总到子文档的分数中。 |
| `inner_hits`      | 可选      | 如果提供，则返回与查询匹配的底层命中（父文档）。                                                                                                                                                                                           |

## 排序限制

`has_parent` 查询不支持使用标准排序选项对结果进行排序。如果您需要按父文档中的字段对子文档进行排序，可以使用 `function_score` 查询并按子文档的得分进行排序。

对于前面的示例，首先添加一个 `customer_satisfaction` 字段，通过该字段对属于父（品牌）文档的子文档进行排序：

```
PUT testindex1/_doc/1
{
  "name": "Luxury watch brand",
  "product_to_brand" : "brand",
  "customer_satisfaction": 4.5
}

PUT testindex1/_doc/2
{
  "name": "Economy watch brand",
  "product_to_brand" : "brand",
  "customer_satisfaction": 3.9
}
```

现在您可以根据父品牌的 `customer_satisfaction` 字段对子文档（产品）进行排序。此查询将分数乘以父文档的 `customer_satisfaction` 字段：

```
GET testindex1/_search
{
  "query": {
    "has_parent": {
      "parent_type": "brand",
      "score": true,
      "query": {
        "function_score": {
          "script_score": {
            "script": "_score * doc['customer_satisfaction'].value"
          }
        }
      }
    }
  }
}
```

包含按最高父级 `customer_satisfaction` 排序的产品：

```
{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 4.5,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "3",
        "_score": 4.5,
        "_routing": "1",
        "_source": {
          "name": "Mechanical watch",
          "sales_count": 150,
          "product_to_brand": {
            "name": "product",
            "parent": "1"
          }
        }
      },
      {
        "_index": "testindex1",
        "_id": "4",
        "_score": 3.9,
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
        "_score": 3.9,
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
```


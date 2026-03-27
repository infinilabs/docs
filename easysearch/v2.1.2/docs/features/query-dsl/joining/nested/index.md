---
title: "Nested 查询"
date: 0001-01-01
summary: "Nested 查询 #  nested 查询充当其他查询的包装器，用于搜索嵌套字段。嵌套字段对象被视为单独的文档进行搜索。如果对象匹配搜索条件，nested 查询将返回根级别的父文档。
相关指南（先读这些） #    Nested 建模  关联查询（Joining）  参考样例 #  在运行 nested 查询之前，您的索引必须包含一个嵌套字段。
要配置一个包含嵌套字段的示例索引，请发送以下请求：
PUT /testindex { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;patient&#34;: { &#34;type&#34;: &#34;nested&#34;, &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;age&#34;: { &#34;type&#34;: &#34;integer&#34; } } } } } } 接下来，将一个文档索引到示例索引中：
PUT /testindex/_doc/1 { &#34;patient&#34;: { &#34;name&#34;: &#34;John Doe&#34;, &#34;age&#34;: 56 } } 要搜索嵌套的 patient 字段，请将您的查询包裹在 nested 查询中，并提供 path 给嵌套字段："
---


# Nested 查询

`nested` 查询充当其他查询的包装器，用于搜索嵌套字段。嵌套字段对象被视为单独的文档进行搜索。如果对象匹配搜索条件，`nested` 查询将返回根级别的父文档。

## 相关指南（先读这些）

- [Nested 建模]({{< relref "/docs/best-practices/data-modeling/nested.md" >}})
- [关联查询（Joining）]({{< relref "/docs/features/query-dsl/joining/_index.md" >}})

## 参考样例

在运行 `nested` 查询之前，您的索引必须包含一个嵌套字段。

要配置一个包含嵌套字段的示例索引，请发送以下请求：

```
PUT /testindex
{
  "mappings": {
    "properties": {
      "patient": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "age": {
            "type": "integer"
          }
        }
      }
    }
  }
}
```

接下来，将一个文档索引到示例索引中：

```
PUT /testindex/_doc/1
{
  "patient": {
    "name": "John Doe",
    "age": 56
  }
}
```

要搜索嵌套的 `patient` 字段，请将您的查询包裹在 `nested` 查询中，并提供 `path` 给嵌套字段：

```
GET /testindex/_search
{
  "query": {
    "nested": {
      "path": "patient",
      "query": {
        "match": {
          "patient.name": "John"
        }
      }
    }
  }
}
```

查询返回匹配的文档：

```
{
  "took": 3,
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
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "patient": {
            "name": "John Doe",
            "age": 56
          }
        }
      }
    ]
  }
}

```

## 检索内部命中项

要返回与查询匹配的内部命中，请提供 `inner_hits` 参数：

```
GET /testindex/_search
{
  "query": {
    "nested": {
      "path": "patient",
      "query": {
        "match": {
          "patient.name": "John"
        }
      },
      "inner_hits": {}
    }
  }
}
```

结果包含额外的 `inner_hits` 字段。 `_nested` 字段标识了内部命中来源的具体内部对象。它包含嵌套命中及其相对于 `_source` 中位置的偏移量。由于排序和评分， `inner_hits` 中命中对象的位置通常与其在嵌套对象中的原始位置不同。

默认情况下，返回的击中对象中的 `_source` 相对于 `inner_hits` 字段。在这个例子中， `inner_hits` 中的 `_source` 包含 `name` 和 `age` 字段，而不是顶层的 `_source` ，后者包含整个 `patient` 对象：

```
{
  "took": 38,
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
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "patient": {
            "name": "John Doe",
            "age": 56
          }
        },
        "inner_hits": {
          "patient": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 0.2876821,
              "hits": [
                {
                  "_index": "testindex",
                  "_id": "1",
                  "_nested": {
                    "field": "patient",
                    "offset": 0
                  },
                  "_score": 0.2876821,
                  "_source": {
                    "name": "John Doe",
                    "age": 56
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

## 多级嵌套查询

您可以使用多级嵌套查询来搜索包含在其他嵌套对象内部的嵌套对象。在这个例子中，您将通过为层次结构的每一级指定嵌套查询来查询多个层级的嵌套字段。

首先，创建一个具有多级嵌套字段的索引：

```
PUT /patients
{
  "mappings": {
    "properties": {
      "patient": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "contacts": {
            "type": "nested",
            "properties": {
              "name": {
                "type": "text"
              },
              "relationship": {
                "type": "text"
              },
              "phone": {
                "type": "keyword"
              }
            }
          }
        }
      }
    }
  }
}
```

接下来，将一个文档索引到示例索引中：

```
PUT /patients/_doc/1
{
  "patient": {
    "name": "John Doe",
    "contacts": [
      {
        "name": "Jane Doe",
        "relationship": "mother",
        "phone": "5551111"
      },
      {
        "name": "Joe Doe",
        "relationship": "father",
        "phone": "5552222"
      }
    ]
  }
}
```

要搜索嵌套的 `patient` 字段，请使用多级 `nested` 查询。以下查询搜索联系信息中包含名为 `Jane` 且关系为 `mother` 的人的病人：

```
GET /patients/_search
{
  "query": {
    "nested": {
      "path": "patient",
      "query": {
        "nested": {
          "path": "patient.contacts",
          "query": {
            "bool": {
              "must": [
                { "match": { "patient.contacts.relationship": "mother" } },
                { "match": { "patient.contacts.name": "Jane" } }
              ]
            }
          }
        }
      }
    }
  }
}
```

查询返回具有匹配这些详细信息的联系记录的患者：

```
{
  "took": 14,
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
    "max_score": 1.3862942,
    "hits": [
      {
        "_index": "patients",
        "_id": "1",
        "_score": 1.3862942,
        "_source": {
          "patient": {
            "name": "John Doe",
            "contacts": [
              {
                "name": "Jane Doe",
                "relationship": "mother",
                "phone": "5551111"
              },
              {
                "name": "Joe Doe",
                "relationship": "father",
                "phone": "5552222"
              }
            ]
          }
        }
      }
    ]
  }
}

```

## 参数说明

以下表格列出了所有由 `nested` 查询支持的一级参数。

| 参数              | 必填/可选 | 描述                                                                                                                                                                                                                                                                                                                                                                    |
| ----------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path`            | 必填      | 指定要搜索的嵌套对象的路径。                                                                                                                                                                                                                                                                                                                                            |
| `query`           | 必填      | 在指定的 path 中运行的查询。如果一个嵌套对象与查询匹配，则返回根父文档。您可以使用点符号搜索嵌套字段，例如 nested_object.subfield 。支持多级嵌套，并自动检测。因此，另一个嵌套查询内的内部 nested 查询会自动匹配正确的嵌套级别，而不是根级别。                                                                                                                          |
| `ignore_unmapped	` | 可选      | 指示是否忽略未映射的 path 字段，而不是抛出错误而不返回文档。您可以在查询多个索引时提供此参数，其中一些索引可能不包含 path 字段。默认为 false 。                                                                                                                                                                                                                         |
| `score_mode`      | 可选      | 定义匹配的内部文档的分数如何影响父文档的分数。有效值有：<br>- avg : 使用所有匹配的内部文档的平均相关性分数。<br>- max : 将匹配的内部文档中的最高相关性分数分配给父文档。<br>- min : 将匹配的内部文档中的最低相关性分数分配给父文档。<br>- sum : 将所有匹配的内部文档的相关性得分相加。<br>- none : 忽略内部文档的相关性得分，并将得分 0 分配给父文档。<br>默认为 avg 。 |
| `inner_hits`      | 可选      | 如果提供，则返回与查询匹配的底层命中。                                                                                                                                                                                                                                                                                                                                  |


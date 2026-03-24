---
title: "Percolate 查询"
date: 0001-01-01
summary: "Percolate 查询 #  使用 percolate 查询来查找与给定文档匹配的已存储查询。此操作与常规搜索相反：常规搜索是查找与查询匹配的文档，而 percolate 查询是查找与文档匹配的查询。percolate 查询通常用于告警、通知和反向搜索用例。
相关指南（先读这些） #    Query DSL 基础  专业查询（Specialized queries）  在使用 percolate 查询时，请考虑以下几点：
 您可以在线提供文档进行 percolate 操作，或者从索引中获取现有文档。 文档和存储的查询必须使用相同的字段名称和类型。 您可以结合使用透查、过滤和评分来构建复杂的匹配系统。 percolate 查询被视为昂贵的查询，并且只有在集群设置 search.allow_expensive_queries 被设置为 true （默认值）时才会运行。如果此设置是 false ， percolate 查询将被拒绝。  percolate 查询在各种实时匹配场景中非常有用。一些常见的用例包括：
 电子商务通知：用户可以注册对产品的兴趣，例如，“有新的苹果笔记本电脑时通知我”。当新产品文档被索引时，系统会找到所有匹配保存的查询的用户并发送警报。 工作警报：求职者根据首选的工作标题或地点保存查询，新的职位发布将与这些查询匹配以触发警报。 安全和警报系统：Percolate 传入的日志或事件数据与保存的规则或异常模式进行匹配。 新闻筛选：将传入的文章与保存的主题配置文件进行匹配，以分类或提供相关内容。  工作过程 #   保存的查询存储在一个特殊的 percolator 字段类型中。 文档会与所有保存的查询进行比较。 每个匹配的查询都会返回其 _id 。 如果启用了高亮显示，匹配的文本片段也会被返回。 如果发送了多个文档， _percolator_document_slot 会显示匹配的文档。  参考样例 #  以下示例展示了如何存储 percolate 查询，并使用不同方法对它们进行测试。"
---


# Percolate 查询

使用 `percolate` 查询来查找与给定文档匹配的已存储查询。此操作与常规搜索相反：常规搜索是查找与查询匹配的文档，而 `percolate` 查询是查找与文档匹配的查询。`percolate` 查询通常用于告警、通知和反向搜索用例。

## 相关指南（先读这些）

- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [专业查询（Specialized queries）]({{< relref "/docs/features/query-dsl/specialized/_index.md" >}})

在使用 `percolate` 查询时，请考虑以下几点：

- 您可以在线提供文档进行 percolate 操作，或者从索引中获取现有文档。
- 文档和存储的查询必须使用相同的字段名称和类型。
- 您可以结合使用透查、过滤和评分来构建复杂的匹配系统。
- `percolate` 查询被视为昂贵的查询，并且只有在集群设置 `search.allow_expensive_queries` 被设置为 `true` （默认值）时才会运行。如果此设置是 `false` ， `percolate` 查询将被拒绝。

`percolate` 查询在各种实时匹配场景中非常有用。一些常见的用例包括：

- 电子商务通知：用户可以注册对产品的兴趣，例如，“有新的苹果笔记本电脑时通知我”。当新产品文档被索引时，系统会找到所有匹配保存的查询的用户并发送警报。
- 工作警报：求职者根据首选的工作标题或地点保存查询，新的职位发布将与这些查询匹配以触发警报。
- 安全和警报系统：Percolate 传入的日志或事件数据与保存的规则或异常模式进行匹配。
- 新闻筛选：将传入的文章与保存的主题配置文件进行匹配，以分类或提供相关内容。

## 工作过程

1. 保存的查询存储在一个特殊的 `percolator` 字段类型中。
2. 文档会与所有保存的查询进行比较。
3. 每个匹配的查询都会返回其 `_id` 。
4. 如果启用了高亮显示，匹配的文本片段也会被返回。
5. 如果发送了多个文档， `_percolator_document_slot` 会显示匹配的文档。

## 参考样例

以下示例展示了如何存储 `percolate` 查询，并使用不同方法对它们进行测试。

首先，创建一个索引并配置其 `mappings` 字段类型以存储保存的查询：

```
PUT /my_percolator_index
{
  "mappings": {
    "properties": {
      "query": {
        "type": "percolator"
      },
      "title": {
        "type": "text"
      }
    }
  }
}
```

在 `title` 字段中添加一个匹配“apple”的查询：

```
POST /my_percolator_index/_doc/1
{
  "query": {
    "match": {
      "title": "apple"
    }
  }
}
```

在 `title` 字段中添加一个匹配“banana”的查询：

```
POST /my_percolator_index/_doc/2
{
  "query": {
    "match": {
      "title": "banana"
    }
  }
}
```

### 对内联文档进行渗透

测试内联文档与保存的查询进行比对：

```
POST /my_percolator_index/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "title": "Fresh Apple Harvest"
      }
    }
  }
}
```

返回结果提供了存储的 `percolate` 查询，该查询用于搜索包含“apple”一词的 `title` 字段，由 `_id` 标识： 1 ：

```
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.13076457,
    "hits": [
      {
        "_index": "my_percolator_index",
        "_id": "1",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "title": "apple"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        }
      }
    ]
  }
}
```

### 使用多个文档进行渗透

要在同一查询中测试多个文档，请使用以下请求：

```
POST /my_percolator_index/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "documents": [
        { "title": "Banana flavoured ice-cream" },
        { "title": "Apple pie recipe" },
        { "title": "Banana bread instructions" },
        { "title": "Cherry tart" }
      ]
    }
  }
}
```

`_percolator_document_slot` 字段帮助您通过索引识别与每个保存的查询匹配的每个文档：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.54726034,
    "hits": [
      {
        "_index": "my_percolator_index",
        "_id": "1",
        "_score": 0.54726034,
        "_source": {
          "query": {
            "match": {
              "title": "apple"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            1
          ]
        }
      },
      {
        "_index": "my_percolator_index",
        "_id": "2",
        "_score": 0.31506687,
        "_source": {
          "query": {
            "match": {
              "title": "banana"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0,
            2
          ]
        }
      }
    ]
  }
}

```

### 为现有索引文档进行渗透

您可以引用已存储在另一个索引中的现有文档，以检查匹配 `percolate` 查询。

为您的文档创建一个单独的索引：

```
PUT /products
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```

添加文档：

```
POST /products/_doc/1
{
  "title": "Banana Smoothie Special"
}
```

检查存储的查询是否与索引文档匹配：

```
POST /my_percolator_index/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "index": "products",
      "id": "1"
    }
  }
}
```

> 使用存储文档时，你必须同时提供 `index` 和 `id` 。

返回相应的查询：

```
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.13076457,
    "hits": [
      {
        "_index": "my_percolator_index",
        "_id": "2",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "title": "banana"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        }
      }
    ]
  }
}

```

### 批量渗透（多个文档）

您可以在一个请求中检查多个文档：

```
POST /my_percolator_index/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "documents": [
        { "title": "Apple event coming soon" },
        { "title": "Banana farms expand" },
        { "title": "Cherry season starts" }
      ]
    }
  }
}
```

每个匹配项都表示匹配的文档，位于 `_percolator_document_slot` 字段中：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.46484798,
    "hits": [
      {
        "_index": "my_percolator_index",
        "_id": "2",
        "_score": 0.46484798,
        "_source": {
          "query": {
            "match": {
              "title": "banana"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            1
          ]
        }
      },
      {
        "_index": "my_percolator_index",
        "_id": "1",
        "_score": 0.41211313,
        "_source": {
          "query": {
            "match": {
              "title": "apple"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        }
      }
    ]
  }
}

```

### 使用命名查询进行多查询透传

您可以在命名查询中透传不同的文档：

```
GET /my_percolator_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "percolate": {
            "field": "query",
            "document": {
              "title": "Apple pie recipe"
            },
            "name": "apple_doc"
          }
        },
        {
          "percolate": {
            "field": "query",
            "document": {
              "title": "Banana bread instructions"
            },
            "name": "banana_doc"
          }
        }
      ]
    }
  }
}
```

参数 `name` 被附加到 `_percolator_document_slot` 以提供匹配的查询：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.13076457,
    "hits": [
      {
        "_index": "my_percolator_index",
        "_id": "1",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "title": "apple"
            }
          }
        },
        "fields": {
          "_percolator_document_slot_apple_doc": [
            0
          ]
        }
      },
      {
        "_index": "my_percolator_index",
        "_id": "2",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "title": "banana"
            }
          }
        },
        "fields": {
          "_percolator_document_slot_banana_doc": [
            0
          ]
        }
      }
    ]
  }
}

```

这种方法使您能够为单个文档配置更自定义的查询逻辑。在以下示例中，第一个文档查询 `title` 字段，第二个文档查询 `description` 字段。还提供了 `boost` 参数：

```
GET /my_percolator_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "constant_score": {
            "filter": {
              "percolate": {
                "field": "query",
                "document": {
                  "title": "Apple pie recipe"
                },
                "name": "apple_doc"
              }
            },
            "boost": 1.0
          }
        },
        {
          "constant_score": {
            "filter": {
              "percolate": {
                "field": "query",
                "document": {
                  "description": "Banana bread with honey"
                },
                "name": "banana_doc"
              }
            },
            "boost": 3.0
          }
        }
      ]
    }
  }
}
```

## 批量渗透与命名渗透比较

批量渗透（使用 `documents` ）和命名渗透（使用 `bool` 与 `name` ）均可用于渗透多个文档，但它们在结果标签、解释和控制方式上有所不同。它们提供功能相似的结果，但在结构上有重要差异，如下表所述。

| 特性               | 批量（ documents ）              | 命名（ bool + percolate + name ）         |
| ------------------ | -------------------------------- | ----------------------------------------- |
| 输入格式           | 每个 percolate 子句，文档数组    | 多个 percolate 子句，每个文档一个         |
| 每个文档的可追溯性 | 通过槽位索引（0，1，…）          | 通过名称（ `apple_doc` ， `banana_doc` ） |
| 匹配槽位的响应字段 | `_percolator_document_slot: [0]` | `_percolator_document_slot_<name>: [0]`   |
| 高亮前缀           | `0_title`, `1_title`             | `apple_doc_title`, `banana_doc_title`     |
| 每个文档自定义控制 | 不支持                           | 可自定义每个子句                          |
| 支持提升和过滤器   | 不                               | 是（每条子句）                            |
| 性能               | 适用于大批量                     | 子句较多时稍慢                            |
| 用例               | 批量匹配任务，大型事件流         | 单文档跟踪、测试、自定义控制              |

## 高亮匹配

`percolate` 查询与常规查询处理高亮的方式不同：

- 在常规查询中，文档存储在索引中，搜索查询用于高亮匹配的词项。
- 在 `percolate` 查询中，角色被反转：保存的查询（在 `percolator` 索引中）用于高亮文档。

这意味着在 `document` 或 `documents` 中提供的文档是高亮的靶标，而 `percolate` 查询决定了要高亮的区域。

### 突出显示单个文档

此示例使用 `my_percolator_index` 中先前定义的搜索。使用以下请求突出显示 `title` 字段中的匹配项：

```
POST /my_percolator_index/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "title": "Apple banana smoothie"
      }
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```

匹配项根据匹配的查询突出显示：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.13076457,
    "hits": [
      {
        "_index": "my_percolator_index",
        "_id": "1",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "title": "apple"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        },
        "highlight": {
          "title": [
            "<em>Apple</em> banana smoothie"
          ]
        }
      },
      {
        "_index": "my_percolator_index",
        "_id": "2",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "title": "banana"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        },
        "highlight": {
          "title": [
            "Apple <em>banana</em> smoothie"
          ]
        }
      }
    ]
  }
}

```

### 突出显示多个文档

使用 `documents` 数组对多个文档进行透化时，每个文档会被分配一个槽索引。高亮键的格式如下，其中 `<slot> `是你的 `documents` 数组中文档的索引：

```
"<slot>_<fieldname>": [ ... ]

```

使用以下命令对两个文档进行透化并高亮显示：

```
POST /my_percolator_index/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "documents": [
        { "title": "Apple pie recipe" },
        { "title": "Banana smoothie ideas" }
      ]
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```

结果包含以文档槽为前缀的高亮字段，例如 `0_title` 和 `1_title` ：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.31506687,
    "hits": [
      {
        "_index": "my_percolator_index",
        "_id": "1",
        "_score": 0.31506687,
        "_source": {
          "query": {
            "match": {
              "title": "apple"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        },
        "highlight": {
          "0_title": [
            "<em>Apple</em> pie recipe"
          ]
        }
      },
      {
        "_index": "my_percolator_index",
        "_id": "2",
        "_score": 0.31506687,
        "_source": {
          "query": {
            "match": {
              "title": "banana"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            1
          ]
        },
        "highlight": {
          "1_title": [
            "<em>Banana</em> smoothie ideas"
          ]
        }
      }
    ]
  }
}

```

## 参数说明

`percolate` 查询支持以下参数。

| 参数         | 必需/可选 | 描述                                                                                 |
| ------------ | --------- | ------------------------------------------------------------------------------------ |
| `field`      | 必需      | 包含存储的 `percolate` 查询的字段。                                                  |
| `document`   | 可选      | 用于匹配已保存查询的单个内联文档。                                                   |
| `documents`  | 可选      | 用于匹配已保存查询的多个内联文档数组。                                               |
| `index`      | 可选      | 包含要匹配的文档的索引。                                                             |
| `id`         | 可选      | 从索引中获取文档的 ID。                                                              |
| `routing`    | 可选      | 获取文档时使用的路由值。                                                             |
| `preference` | 可选      | 获取文档时对分片路由的偏好设置。                                                     |
| `name`       | 可选      | 为 `percolate` 子句指定的名称。在 `bool` 查询中使用多个 `percolate` 子句时很有帮助。 |


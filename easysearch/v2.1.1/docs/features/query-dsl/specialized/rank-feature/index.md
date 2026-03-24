---
title: "Rank Feature 查询"
date: 0001-01-01
summary: "Rank Feature 查询 #  使用 rank_feature 查询根据文档中的数值（如相关性分数、人气或新鲜度）提升文档分数。如果你希望使用数值特征微调相关性排名，这种查询非常理想。与全文检索不同，rank_feature 仅关注数值信号；在复合查询（如 bool）中与其他查询结合时效果最佳。
rank_feature 查询要求目标字段映射为 rank_feature 字段类型。这可以启用内部优化的评分，从而实现快速高效的提升。
相关指南（先读这些） #    相关性与打分策略  Query DSL 基础  分数影响取决于字段值以及可选的 saturation 、 log 或 sigmoid 函数。这些函数在查询时动态应用以计算最终文档分数；它们不会更改或存储文档中的任何值。
参数说明 #  rank_feature 查询支持以下参数。
   参数 数据类型 必需/可选 描述     field String 必需 一个 rank_feature 或 rank_features 字段，用于影响文档评分。   boost Float 可选 应用于评分的乘数。默认值为 1.0 。0 到 1 之间的值会降低评分；大于 1 的值会提高评分。   saturation Object 可选 对特征值应用饱和函数。随着值的增加，增益会增长，但在 pivot 之后会趋于平稳。如果没有提供其他函数，则使用此默认函数。一次只能使用 saturation 、 log 或 sigmoid 中的一个函数。   log Object 可选 使用基于字段值的对数评分函数。适用于大范围值的场景。一次只能使用 saturation 、 log 或 sigmoid 中的一个函数。   sigmoid Object 可选 对评分影响应用 S 形曲线，由 pivot 和 exponent 控制。一次只能使用 saturation 、 log 或 sigmoid 中的一个函数。   positive_score_impact Boolean 可选 当 false 时，较低值会获得更高的评分。适用于像价格这样的特征，其中较小值更好。作为映射的一部分定义。默认值为 true 。    参考样例 #  以下示例展示了如何定义和使用 rank_feature 字段来影响文档评分。"
---


# Rank Feature 查询

使用 `rank_feature` 查询根据文档中的数值（如相关性分数、人气或新鲜度）提升文档分数。如果你希望使用数值特征微调相关性排名，这种查询非常理想。与全文检索不同，`rank_feature` 仅关注数值信号；在复合查询（如 `bool`）中与其他查询结合时效果最佳。

`rank_feature` 查询要求目标字段映射为 `rank_feature` 字段类型。这可以启用内部优化的评分，从而实现快速高效的提升。

## 相关指南（先读这些）

- [相关性与打分策略]({{< relref "/docs/features/fulltext-search/relevance/_index.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

分数影响取决于字段值以及可选的 `saturation` 、 `log` 或 `sigmoid` 函数。这些函数在查询时动态应用以计算最终文档分数；它们不会更改或存储文档中的任何值。

## 参数说明

`rank_feature` 查询支持以下参数。

| 参数                    | 数据类型 | 必需/可选 | 描述                                                                                                                                                                        |
| ----------------------- | -------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`                 | String   | 必需      | 一个 rank_feature 或 rank_features 字段，用于影响文档评分。                                                                                                                 |
| `boost`                 | Float    | 可选      | 应用于评分的乘数。默认值为 1.0 。0 到 1 之间的值会降低评分；大于 1 的值会提高评分。                                                                                         |
| `saturation`            | Object   | 可选      | 对特征值应用饱和函数。随着值的增加，增益会增长，但在 pivot 之后会趋于平稳。如果没有提供其他函数，则使用此默认函数。一次只能使用 saturation 、 log 或 sigmoid 中的一个函数。 |
| `log`                   | Object   | 可选      | 使用基于字段值的对数评分函数。适用于大范围值的场景。一次只能使用 saturation 、 log 或 sigmoid 中的一个函数。                                                                |
| `sigmoid`               | Object   | 可选      | 对评分影响应用 S 形曲线，由 pivot 和 exponent 控制。一次只能使用 saturation 、 log 或 sigmoid 中的一个函数。                                                                |
| `positive_score_impact` | Boolean  | 可选      | 当 false 时，较低值会获得更高的评分。适用于像价格这样的特征，其中较小值更好。作为映射的一部分定义。默认值为 true 。                                                         |

## 参考样例

以下示例展示了如何定义和使用 `rank_feature` 字段来影响文档评分。

定义一个包含 `rank_feature` 字段的索引来表示信号，如 `popularity` :

```
PUT /products
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "popularity": { "type": "rank_feature" }
    }
  }
}
```

添加具有不同流行度值的示例产品：

```
POST /products/_bulk
{ "index": { "_id": 1 } }
{ "title": "Wireless Earbuds", "popularity": 1 }
{ "index": { "_id": 2 } }
{ "title": "Bluetooth Speaker", "popularity": 10 }
{ "index": { "_id": 3 } }
{ "title": "Portable Charger", "popularity": 25 }
{ "index": { "_id": 4 } }
{ "title": "Smartwatch", "popularity": 50 }
{ "index": { "_id": 5 } }
{ "title": "Noise Cancelling Headphones", "popularity": 100 }
{ "index": { "_id": 6 } }
{ "title": "Gaming Laptop", "popularity": 250 }
{ "index": { "_id": 7 } }
{ "title": "4K Monitor", "popularity": 500 }
```

### 基础排序功能查询

你可以使用 `rank_feature` 基于 `popularity` 分数来提升结果：

```
POST /products/_search
{
  "query": {
    "rank_feature": {
      "field": "popularity"
    }
  }
}
```

这个查询本身不执行过滤。相反，它根据 `popularity` 的值对所有文档进行评分：更高的值会产生更高的分数：

```
{
  ...
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": 0.9252834,
    "hits": [
      {
        "_index": "products",
        "_id": "7",
        "_score": 0.9252834,
        "_source": {
          "title": "4K Monitor",
          "popularity": 500
        }
      },
      {
        "_index": "products",
        "_id": "6",
        "_score": 0.86095566,
        "_source": {
          "title": "Gaming Laptop",
          "popularity": 250
        }
      },
      {
        "_index": "products",
        "_id": "5",
        "_score": 0.71237755,
        "_source": {
          "title": "Noise Cancelling Headphones",
          "popularity": 100
        }
      },
      {
        "_index": "products",
        "_id": "4",
        "_score": 0.5532503,
        "_source": {
          "title": "Smartwatch",
          "popularity": 50
        }
      },
      {
        "_index": "products",
        "_id": "3",
        "_score": 0.38240916,
        "_source": {
          "title": "Portable Charger",
          "popularity": 25
        }
      },
      {
        "_index": "products",
        "_id": "2",
        "_score": 0.19851118,
        "_source": {
          "title": "Bluetooth Speaker",
          "popularity": 10
        }
      },
      {
        "_index": "products",
        "_id": "1",
        "_score": 0.024169207,
        "_source": {
          "title": "Wireless Earbuds",
          "popularity": 1
        }
      }
    ]
  }
}

```

### 与全文搜索结合

要筛选相关结果并根据热度提升它们，请使用以下请求。此查询对匹配“headphones”的所有文档进行排序，并提升那些热度更高的文档：

```
POST /products/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "headphones"
        }
      },
      "should": {
        "rank_feature": {
          "field": "popularity"
        }
      }
    }
  }
}
```

### 权重提升

`boost` 参数允许你调整 `rank_feature` 子句的分数贡献。这在 `bool` 等复合查询中特别有用，你可以控制数值字段（如人气、新鲜度或相关性分数）对最终文档排序的影响程度。

在以下示例中， `bool` 查询匹配包含“headphones”的文档，并使用 `rank_feature` 子句和 `boost` 为 2.0 的 `rank_feature` 子句来提升更受欢迎的结果。这使 `rank_feature` 分数对整体文档分数的贡献翻倍：

```
POST /products/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "headphones"
        }
      },
      "should": {
        "rank_feature": {
          "field": "popularity",
          "boost": 2.0
        }
      }
    }
  }
}
```

### 配置得分函数

默认情况下， `rank_feature` 查询使用一个 `saturation` 函数，其 `pivot` 值由字段派生。您可以显式地将函数设置为 `saturation` 、 `log` 或 `sigmoid` 。

#### 饱和函数

`saturation` 函数是 `rank_feature` 查询中使用的默认得分方法。它为具有较大特征值的文档分配更高的得分，但随着值超过指定的 `pivot` ，得分的增加会变得更加平缓。当您希望对非常大的值给予递减回报时，这很有用，例如，在避免过度奖励极高数字的同时提升 `popularity` 。计算得分的公式是 `value of the rank_feature field / (value of the rank_feature field + pivot)` 。产生的得分始终在 0 和 1 之间。如果未提供 `pivot` ，则使用索引中所有 `rank_feature` 值的近似几何平均值。

以下示例使用 `saturation` ，其 `pivot` 为 50 ：

```
POST /products/_search
{
  "query": {
    "rank_feature": {
      "field": "popularity",
      "saturation": {
        "pivot": 50
      }
    }
  }
}
```

`pivot` 定义了评分增长减缓的点。大于 `pivot` 的值仍然会增加评分，但会随着收益递减，如返回的命中所示：

```
{
  ...
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": 0.9090909,
    "hits": [
      {
        "_index": "products",
        "_id": "7",
        "_score": 0.9090909,
        "_source": {
          "title": "4K Monitor",
          "popularity": 500
        }
      },
      {
        "_index": "products",
        "_id": "6",
        "_score": 0.8333333,
        "_source": {
          "title": "Gaming Laptop",
          "popularity": 250
        }
      },
      {
        "_index": "products",
        "_id": "5",
        "_score": 0.6666666,
        "_source": {
          "title": "Noise Cancelling Headphones",
          "popularity": 100
        }
      },
      {
        "_index": "products",
        "_id": "4",
        "_score": 0.5,
        "_source": {
          "title": "Smartwatch",
          "popularity": 50
        }
      },
      {
        "_index": "products",
        "_id": "3",
        "_score": 0.3333333,
        "_source": {
          "title": "Portable Charger",
          "popularity": 25
        }
      },
      {
        "_index": "products",
        "_id": "2",
        "_score": 0.16666669,
        "_source": {
          "title": "Bluetooth Speaker",
          "popularity": 10
        }
      },
      {
        "_index": "products",
        "_id": "1",
        "_score": 0.019607842,
        "_source": {
          "title": "Wireless Earbuds",
          "popularity": 1
        }
      }
    ]
  }
}

```

#### LOG 函数

当 `rank_feature` 字段包含一个显著范围内的值时， `log` 函数很有用。它对 `score` 应用对数刻度，减少极高值的影响，并帮助在宽泛的值分布中实现评分的归一化。当低值之间的小差异应该比高值之间的大差异更有影响时，这尤其有用。评分使用公式 `log(scaling_factor + rank_feature field)` 计算。以下示例使用 `scaling_factor` 为 2 ：

```
POST /products/_search
{
  "query": {
    "rank_feature": {
      "field": "popularity",
      "log": {
        "scaling_factor": 2
      }
    }
  }
}
```

在示例数据集中， `popularity` 字段的范围从 1 到 500 。 `log` 函数压缩了来自 250 和 500 等大值的 `score` 贡献，同时仍然允许具有 10 或 25 的文档获得有意义的分数。相比之下，如果你应用 `saturation` 函数，高于 `pivot` 的文档会迅速接近相同的最大分数：

```
{
  ...
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": 6.2186003,
    "hits": [
      {
        "_index": "products",
        "_id": "7",
        "_score": 6.2186003,
        "_source": {
          "title": "4K Monitor",
          "popularity": 500
        }
      },
      {
        "_index": "products",
        "_id": "6",
        "_score": 5.529429,
        "_source": {
          "title": "Gaming Laptop",
          "popularity": 250
        }
      },
      {
        "_index": "products",
        "_id": "5",
        "_score": 4.624973,
        "_source": {
          "title": "Noise Cancelling Headphones",
          "popularity": 100
        }
      },
      {
        "_index": "products",
        "_id": "4",
        "_score": 3.9512436,
        "_source": {
          "title": "Smartwatch",
          "popularity": 50
        }
      },
      {
        "_index": "products",
        "_id": "3",
        "_score": 3.295837,
        "_source": {
          "title": "Portable Charger",
          "popularity": 25
        }
      },
      {
        "_index": "products",
        "_id": "2",
        "_score": 2.4849067,
        "_source": {
          "title": "Bluetooth Speaker",
          "popularity": 10
        }
      },
      {
        "_index": "products",
        "_id": "1",
        "_score": 1.0986123,
        "_source": {
          "title": "Wireless Earbuds",
          "popularity": 1
        }
      }
    ]
  }
}

```

#### SIGMOID 函数

`sigmoid` 函数提供了一个平滑的 S 形评分曲线，这在你想控制评分影响的陡峭程度和中点时特别有用。评分使用公式 `rank feature field value^exp / (rank feature field value^exp + pivot^exp)` 计算得出。以下示例使用了一个配置了 `pivot` 和 `exponent` 的 `sigmoid` 函数。 `pivot` 定义了评分等于 0.5 的值。 `exponent` 控制曲线的陡峭程度。较低值会导致在 `pivot` 周围的过渡更加尖锐：

```
POST /products/_search
{
  "query": {
    "rank_feature": {
      "field": "popularity",
      "sigmoid": {
        "pivot": 50,
        "exponent": 0.5
      }
    }
  }
}
```

`sigmoid` 函数平滑地提升了 `pivot` （在本例中为 50 ）周围的分数，对接近 `pivot` 的值给予适度偏好，同时使高低两端的分数趋于平缓：

```
{
  ...
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": 0.7597469,
    "hits": [
      {
        "_index": "products",
        "_id": "7",
        "_score": 0.7597469,
        "_source": {
          "title": "4K Monitor",
          "popularity": 500
        }
      },
      {
        "_index": "products",
        "_id": "6",
        "_score": 0.690983,
        "_source": {
          "title": "Gaming Laptop",
          "popularity": 250
        }
      },
      {
        "_index": "products",
        "_id": "5",
        "_score": 0.58578646,
        "_source": {
          "title": "Noise Cancelling Headphones",
          "popularity": 100
        }
      },
      {
        "_index": "products",
        "_id": "4",
        "_score": 0.5,
        "_source": {
          "title": "Smartwatch",
          "popularity": 50
        }
      },
      {
        "_index": "products",
        "_id": "3",
        "_score": 0.41421357,
        "_source": {
          "title": "Portable Charger",
          "popularity": 25
        }
      },
      {
        "_index": "products",
        "_id": "2",
        "_score": 0.309017,
        "_source": {
          "title": "Bluetooth Speaker",
          "popularity": 10
        }
      },
      {
        "_index": "products",
        "_id": "1",
        "_score": 0.12389934,
        "_source": {
          "title": "Wireless Earbuds",
          "popularity": 1
        }
      }
    ]
  }
}
```

#### 分数反转

默认情况下，较高的值会导致较高的分数。如果您希望较低的值产生较高的分数（例如，较低的价格更相关），请在索引创建期间将 `positive_score_impact` 设置为 `false` ：

```
PUT /products_new
{
  "mappings": {
    "properties": {
      "popularity": {
        "type": "rank_feature",
        "positive_score_impact": false
      }
    }
  }
}
```


---
title: "Function Score 查询"
date: 0001-01-01
summary: "Function Score 查询 #  如果您需要更改结果中返回的文档的相关性评分，请使用 function_score 查询。function_score 查询定义了一个查询和一个或多个函数，这些函数可以应用于所有结果或结果的一部分，以重新计算它们的相关性评分。
相关指南（先读这些） #    查询 DSL 基础  相关性：加权与调参  使用一个评分函数 #  最基础的 function_score 查询示例使用一个函数来重新计算分数。以下查询使用一个 weight 函数将所有相关性分数加倍。此函数适用于所有结果文档，因为没有在 function_score 中指定 query 参数：
GET shakespeare/_search { &#34;query&#34;: { &#34;function_score&#34;: { &#34;weight&#34;: &#34;2&#34; } } } 将评分函数应用于文档子集 #  要将评分函数应用于文档子集，在函数中提供一个查询：
GET shakespeare/_search { &#34;query&#34;: { &#34;function_score&#34;: { &#34;query&#34;: { &#34;match&#34;: { &#34;play_name&#34;: &#34;Hamlet&#34; } }, &#34;weight&#34;: &#34;2&#34; } } } 支持的功能 #  function_score 查询类型支持以下功能："
---


# Function Score 查询

如果您需要更改结果中返回的文档的相关性评分，请使用 `function_score` 查询。`function_score` 查询定义了一个查询和一个或多个函数，这些函数可以应用于所有结果或结果的一部分，以重新计算它们的相关性评分。

## 相关指南（先读这些）

- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [相关性：加权与调参]({{< relref "/docs/features/fulltext-search/relevance/boosting.md" >}})

## 使用一个评分函数

最基础的 `function_score` 查询示例使用一个函数来重新计算分数。以下查询使用一个 `weight` 函数将所有相关性分数加倍。此函数适用于所有结果文档，因为没有在 `function_score` 中指定 `query` 参数：

```
GET shakespeare/_search
{
  "query": {
    "function_score": {
      "weight": "2"
    }
  }
}
```

## 将评分函数应用于文档子集

要将评分函数应用于文档子集，在函数中提供一个查询：

```
GET shakespeare/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "play_name": "Hamlet"
        }
      },
      "weight": "2"
    }
  }
}
```

## 支持的功能

`function_score` 查询类型支持以下功能：

- 内置:
  - `weight` ：将文档得分乘以一个预定义的增强因子。
  - `random_score` ：提供一个对单个用户一致的随机得分，但不同用户之间得分不同。
  - `field_value_factor` : 使用指定文档字段的值来重新计算分数。
  - 衰减函数（ gauss 、 exp 和 linear ）：使用指定的衰减函数重新计算分数。
- 自定义：
- `script_score` : 使用脚本对文档进行评分。

## 权重函数

当您使用 `weight` 函数时，原始的相关性分数会乘以 `weight` 的浮点值：

```
GET shakespeare/_search
{
  "query": {
    "function_score": {
      "weight": "2"
    }
  }
}
```

与 `boost` 值不同， `weight` 函数没有进行归一化。

## 随机分数函数

`random_score` 函数提供一致的随机分数，但不同用户之间分数不同。该分数是一个位于 [0, 1) 范围内的浮点数。默认情况下， `random_score` 函数使用内部 Lucene 文档 ID 作为种子值，由于合并后文档可以重新编号，因此随机值不可重现。为了在生成随机值时保持一致性，您可以提供 `seed` 和 `field` 参数。 `field` 必须是已启用 `fielddata` 的字段（通常是一个数值字段）。分数是使用 `seed` ， `fielddata` 的值，以及使用索引名称和分片 ID 计算的盐值来计算的。由于索引名称和分片 ID 对于位于同一分片中的文档是相同的，因此具有相同 field 值的文档将被分配相同的分数。为了确保同一分片中的所有文档都有不同的分数，请使用具有所有文档唯一值的 `field` 。一个选项是使用 `_seq_no` 字段。但是，如果您选择此字段，由于相应的 `_seq_no` 更新，分数可能会发生变化。

以下查询使用 `random_score` 函数与 `seed` 和 `field` ：

```
GET blogs/_search
{
  "query": {
    "function_score": {
      "random_score": {
        "seed": 20,
        "field": "_seq_no"
      }
    }
  }
}
```

## 字段值因子函数

`field_value_factor` 函数使用指定文档字段的值重新计算分数。如果该字段是多值字段，则仅使用其第一个值进行计算，其他值不考虑。

`field_value_factor` 函数支持以下选项：

- `field` : 用于评分计算的字段。
- `factor` ：一个可选的系数，用于乘以字段值。默认为 1。
- `modifier` : 应用于字段值 v 的修饰符之一。下表列出了所有支持的修饰符。

| 修改符       | 公式             | 描述                                                                                                                                                                          |
| ------------ | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `log`        | $ \log v $       | 取值的 10 为底的对数。对非正数取对数是非法操作，将导致错误。对于介于 0（不含）和 1（含）之间的值，此函数返回非负值，将导致错误。我们建议使用 `log1p` 或 `log2p` 代替 `log` 。 |
| `log1p`      | $ \log （1+v） $ | 取 1 与值的和的 10 为底的对数。                                                                                                                                               |
| `log2p`      | $ \log （2+v） $ | 取 2 和值的和的 10 为底的对数。                                                                                                                                               |
| `ln`         | $ \ln v $        | 取值的自然对数。对非正数取对数是非法操作，将导致错误。对于介于 0（不含）和 1（含）之间的值，此函数返回非负值，将导致错误。我们建议使用 `ln1p` 或 `ln2p` 代替 `ln` 。          |
| `ln1p`       | $ \ln (1+v) $    | 取 1 和值的和的自然对数。                                                                                                                                                     |
| `reciprocal` | $\frac{1}{v}$    | 取值的倒数。                                                                                                                                                                  |
| `square`     | $v^2$            | 值的平方。                                                                                                                                                                    |
| `sqrt`       | $\sqrt{v}$       | 取值的平方根。对负数取平方根是非法操作，会导致错误。确保 v 是非负数。                                                                                                         |
| `none`       | N/A              | 不应用任何修饰符。                                                                                                                                                            |

- `missing` ：如果字段在文档中缺失时使用的值。 `factor` 和 `modifier` 将应用于此值，而不是缺失字段的值。

例如，以下查询使用 `field_value_factor` 函数给 `views` 字段赋予更高的权重：

```
GET blogs/_search
{
  "query": {
    "function_score": {
      "field_value_factor": {
        "field": "views",
        "factor": 1.5,
        "modifier": "log1p",
        "missing": 1
      }
    }
  }
}
```

前一个查询使用以下公式计算相关性得分：

$\text{score} = \text{original score} \cdot \log(1 + 1.5 \cdot \text{views})$

## 脚本得分函数

使用 `script_score` 函数，您可以编写一个用于评分文档的自定义脚本，可选地结合文档中字段的值。原始的相关性得分可以在 `_score` 变量中访问。

> 计算出的得分不能为负。负得分将导致错误。文档得分具有正的 32 位浮点值。精度更高的得分将转换为最接近的 32 位浮点数。

例如，以下查询使用 `script_score` 函数根据原始分数以及博客文章的观看次数和点赞数来计算分数。为了给观看次数和点赞数赋予较小的权重，此公式取观看次数和点赞数之和的对数。为了使对数在观看次数和点赞数为 0 时仍然有效，将 1 加到它们的和上：

```
GET blogs/_search
{
  "query": {
    "function_score": {
      "query": {"match": {"name": "easysearch"}},
      "script_score": {
        "script": "_score * Math.log(1 + doc['likes'].value + doc['views'].value)"
      }
    }
  }
}
```

脚本被编译并缓存以提高性能。因此，最好重用相同的脚本并传递脚本所需的任何参数：

```
GET blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": { "name": "easysearch" }
      },
      "script_score": {
        "script": {
          "params": {
            "add": 1
          },
          "source": "_score * Math.log(params.add + doc['likes'].value + doc['views'].value)"
        }
      }
    }
  }
}
```

## 衰减函数

对于许多应用，您需要根据邻近度或时间顺序对结果进行排序。您可以使用衰减函数来实现这一点。衰减函数通过三种衰减曲线之一（高斯、指数或线性）计算文档得分。

> 衰减函数仅对数值、日期和地理点字段起作用。

### 示例：地理点字段

假设您正在寻找一家靠近办公室的酒店。您创建一个将 `location` 字段映射为地理点的 `hotels` 索引：

```
PUT hotels
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

您索引了两个与附近酒店对应的文档：

```
PUT hotels/_doc/1
{
  "name": "Hotel Within 200",
  "location": {
    "lat": 40.7105,
    "lon": 74.00
  }
}

PUT hotels/_doc/2
{
  "name": "Hotel Outside 500",
  "location": {
    "lat": 40.7115,
    "lon": 74.00
  }
}
```

`origin` 定义了计算距离的起点（办公室位置）。 `offset` 指定了距离原点一定范围内文档获得满分的距离。您可以给距离办公室 200 英尺内的酒店相同的最高分。 `scale` 定义了图形的衰减率， `decay` 定义了从原点 `scale` + `offset` 距离处分配给文档的分数。一旦超出 200 英尺的半径，您可能决定如果再走 300 英尺才能到达酒店（ `scale` = 300 英尺），则将其分配的分数减为原来的四分之一（ `decay` = 0.25）。

您使用 `origin` 在（74.00，40.71）处创建以下查询：

```
GET hotels/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "exp": {
            "location": {
              "origin": "40.71,74.00",
              "offset": "200ft",
              "scale":  "300ft",
              "decay": 0.25
            }
          }
        }
      ]
    }
  }
}
```

返回内容包含两家酒店。距离办公室 200 英尺内的酒店得分为 1，而距离 500 英尺半径外的酒店得分为 0.20，这低于 `decay` 参数的 0.25：

```
{
  "took": 854,
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
        "_index": "hotels",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Hotel Within 200",
          "location": {
            "lat": 40.7105,
            "lon": 74
          }
        }
      },
      {
        "_index": "hotels",
        "_id": "2",
        "_score": 0.20099315,
        "_source": {
          "name": "Hotel Outside 500",
          "location": {
            "lat": 40.7115,
            "lon": 74
          }
        }
      }
    ]
  }
}
```

### 参数说明

以下表格列出了 `gauss` 、 `exp` 和 `linear` 函数支持的所有参数。

| 参数     | 描述                                                                                                                                                                                                                                                                                                               |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `origin` | 计算距离的起点。对于数值字段，必须提供数字；对于日期字段，必须提供日期；对于地理点字段，必须提供地理点。地理点和数值字段是必需的。对于日期字段是可选的（默认为 now ）。对于日期字段，支持日期计算（例如， now-2d ）。                                                                                              |
| `offset` | 定义了给予文档 1 分的起始距离。可选。默认为 0。                                                                                                                                                                                                                                                                    |
| `scale`  | 距离 scale + offset 的 origin 文档被分配 decay 分。必需。<br> 对于数值字段， scale 可以是任何数字。<br> 对于日期字段， scale 可以定义为带有单位的数字（ 5h ， 1d ）。如果没有提供单位， scale 默认为毫秒。<br>对于地理点字段， scale 可以定义为带有单位的数字（ 1mi ， 5km ）。如果没有提供单位， scale 默认为米。 |
| `decay`  | 定义距离 origin 的 scale + offset 处的文档得分。可选。默认值为 0.5。                                                                                                                                                                                                                                               |

> 对于文档中缺失的字段，衰减函数返回得分为 1。

### 示例：数值字段

以下查询使用指数衰减函数根据评论数量优先排序博客文章：

```
GET blogs/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "exp": {
            "comments": {
              "origin":  "20",
              "offset": "5",
              "scale":  "10"
            }
          }
        }
      ]
    }
  }
}
```

结果中的前两篇博客文章得分为 1，因为一篇位于原点（20），另一篇距离为 16，这处于偏移量内（计算得到满分文档的范围为 20
5，即[15, 25]）。第三篇博客文章距离 `origin` （20 − （5 + 10）= 15）为 `scale` + `offset` ，因此它被赋予默认的 `decay` 得分（0.5）：

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
      "value": 4,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "blogs",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Semantic search in Easysearch",
          "views": 1200,
          "likes": 150,
          "comments": 16,
          "date_posted": "2022-04-17"
        }
      },
      {
        "_index": "blogs",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Get started with Easysearch 2.7",
          "views": 1400,
          "likes": 100,
          "comments": 20,
          "date_posted": "2022-05-02"
        }
      },
      {
        "_index": "blogs",
        "_id": "3",
        "_score": 0.5,
        "_source": {
          "name": "Distributed tracing with Data Prepper",
          "views": 800,
          "likes": 50,
          "comments": 5,
          "date_posted": "2022-04-25"
        }
      },
      {
        "_index": "blogs",
        "_id": "4",
        "_score": 0.4352753,
        "_source": {
          "name": "A very old blog",
          "views": 100,
          "likes": 20,
          "comments": 3,
          "date_posted": "2000-04-25"
        }
      }
    ]
  }
}
```

### 示例：日期字段

以下查询使用高斯衰减函数来优先显示发布于 2002 年 4 月 24 日左右的博客文章：

```
GET blogs/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "gauss": {
            "date_posted": {
              "origin":  "2022-04-24",
              "offset": "1d",
              "scale":  "6d",
              "decay": 0.25
            }
          }
        }
      ]
    }
  }
}

```

在结果中，第一篇博客文章是在 4 月 24 日前后一天发布的，因此它具有最高的分数 1。第二篇博客文章是在 4 月 17 日发布的，它在 `offset` + `scale` （ 1d + 6d ）的范围内，因此得分等于 `decay` （0.25）。第三篇博客文章是在 4 月 24 日之后 7 天以上发布的，因此得分较低。最后一篇博客文章的得分为 0，因为它是在多年前发布的：

```
{
  "took": 2,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "blogs",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Distributed tracing with Data Prepper",
          "views": 800,
          "likes": 50,
          "comments": 5,
          "date_posted": "2022-04-25"
        }
      },
      {
        "_index": "blogs",
        "_id": "1",
        "_score": 0.25,
        "_source": {
          "name": "Semantic search in Easysearch",
          "views": 1200,
          "likes": 150,
          "comments": 16,
          "date_posted": "2022-04-17"
        }
      },
      {
        "_index": "blogs",
        "_id": "2",
        "_score": 0.15154076,
        "_source": {
          "name": "Get started with Easysearch 2.7",
          "views": 1400,
          "likes": 100,
          "comments": 20,
          "date_posted": "2022-05-02"
        }
      },
      {
        "_index": "blogs",
        "_id": "4",
        "_score": 0,
        "_source": {
          "name": "A very old blog",
          "views": 100,
          "likes": 20,
          "comments": 3,
          "date_posted": "2000-04-25"
        }
      }
    ]
  }
}
```

### 多值字段

如果指定的衰减计算字段包含多个值，可以使用 `multi_value_mode` 参数。此参数指定以下函数之一，以确定用于计算的字段值：

- `min` ：（默认）距离 `origin` 的最小距离。
- `max` ：距离 `origin` 的最大距离。
- `avg` ：距离 `origin` 的平均值。
- `sum` ：从 `origin` 计算出的所有距离之和。

例如，您使用一个距离数组索引文档：

```
PUT testindex/_doc/1
{
  "distances": [1, 2, 3, 4, 5]
}
```

以下查询使用多值字段 `distances` 的 `max` 距离来计算衰减：

```
GET testindex/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "exp": {
            "distances": {
              "origin":  "6",
              "offset": "5",
              "scale":  "1"
            },
            "multi_value_mode": "max"
          }
        }
      ]
    }
  }
}
```

由于从原点（1）的最大距离在 `offset` 内，该文档被赋予分数 1：

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
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1,
        "_source": {
          "distances": [
            1,
            2,
            3,
            4,
            5
          ]
        }
      }
    ]
  }
}

```

### 衰减曲线计算

以下公式定义了各种衰减函数的得分计算（ v 表示文档字段值）。

高斯：

$\text{score} = \exp\left(-\frac{(\max(0, |v - \text{origin}| - \text{offset}))^2}{2\sigma^2}\right)$

其中 sigma 计算以确保得分在距离 `origin` 的 `offset` + `scale` 处等于 `decay` ：

$\sigma^2 = -\frac{\text{scale}^2}{2\ln(\text{decay})}$

指数：

$\text{score} = \exp(\lambda \cdot \max(0, |v - \text{origin}| - \text{offset}))$

lambda 是计算以确保在距离 `origin` 的 `offset` + `scale` 处的分数等于 `decay` ：

$\lambda = \frac{\ln(\text{decay})}{\text{scale}}$

线性:

$\text{score} = \max\left( \frac{s - \max(0, |v - \text{origin}| - \text{offset})}{s} \right)$

s 是计算以确保在距离 `origin` 的 `offset` + `scale` 处的分数等于 `decay` ：

$s = \frac{\text{scale}}{1 - \text{decay}}$

## 使用多个评分函数

您可以在函数得分查询中通过在 `functions` 数组中列出它们来指定多个得分函数。

### 组合多个函数的得分

不同的函数可以使用不同的评分尺度。例如， `random_score` 函数提供一个介于 0 和 1 之间的得分，但 `field_value_factor` 没有具体的评分尺度。此外，您可能希望对不同函数给出的得分进行不同的加权。为了调整不同函数的得分，您可以指定每个函数的 `weight` 参数。然后，每个函数给出的得分乘以 `weight` 以产生该函数的最终得分。 `weight` 参数必须在 `functions` 数组中提供，以便与加权函数区分，

每个函数给出的得分通过 `score_mode` 参数进行组合，该参数可以取以下值：

- `multiply` : （默认）分数相乘。
- `sum` : 分数相加。
- `avg` : 分数取平均值。如果指定了 `weight` ，则这是一个加权平均值。例如，如果第一个函数的权重 1 返回分数 10 ，第二个函数的权重 4 返回分数 20 ，则平均值为 18
- `first` : 取第一个匹配过滤器的函数的分数。
- `max` : 取最大分值。
- `min` : 取最小分值。

### 指定分值的上限

您可以在 `max_boost` 参数中指定函数分值的上限。默认上限是 float 值的最大幅度：(2 − 2 ^−23 ) · 2 ^127 。

### 将所有函数的得分与查询得分结合

您可以在 `boost_mode` 参数中指定使用所有函数计算得出的得分如何与查询得分结合，该参数可以取以下值之一：

- `multiply` ：(默认) 将查询得分乘以函数得分。
- `replace` ：忽略查询得分，使用函数得分。
- `sum` : 将查询得分和函数得分相加。
- `avg` : 计算查询得分和函数得分的平均值。
- `max` : 取查询得分和函数得分中的较大值。
- `min` : 取查询得分和函数得分中的较小值。

### 过滤未达到阈值的文档

更改相关性分数不会改变匹配文档的列表。要排除一些未达到阈值的文档，请在 min_score 参数中指定阈值值。然后使用该阈值值对所有查询返回的文档进行评分和过滤。

### 参考样例

以下请求搜索包含“Easysearch Data Prepper”一词的博客文章，优先考虑发布于 2022 年 4 月 24 日左右的帖子。此外，还会考虑查看次数和点赞数。最后，将截止阈值设置为 10 分：

```
GET blogs/_search
{
  "query": {
    "function_score": {
      "boost": "5",
      "functions": [
        {
          "gauss": {
            "date_posted": {
              "origin": "2022-04-24",
              "offset": "1d",
              "scale": "6d"
            }
          },
          "weight": 1
        },
        {
          "gauss": {
            "likes": {
              "origin": 200,
              "scale": 200
            }
          },
          "weight": 4
        },
        {
          "gauss": {
            "views": {
              "origin": 1000,
              "scale": 800
            }
          },
          "weight": 2
        }
      ],
      "query": {
        "match": {
          "name": "easysearch data prepper"
        }
      },
      "max_boost": 10,
      "score_mode": "max",
      "boost_mode": "multiply",
      "min_score": 10
    }
  }
}
```

结果包含三篇匹配的博客文章：

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
      "value": 3,
      "relation": "eq"
    },
    "max_score": 31.191923,
    "hits": [
      {
        "_index": "blogs",
        "_id": "3",
        "_score": 31.191923,
        "_source": {
          "name": "Distributed tracing with Data Prepper",
          "views": 800,
          "likes": 50,
          "comments": 5,
          "date_posted": "2022-04-25"
        }
      },
      {
        "_index": "blogs",
        "_id": "1",
        "_score": 13.907352,
        "_source": {
          "name": "Semantic search in Easysearch",
          "views": 1200,
          "likes": 150,
          "comments": 16,
          "date_posted": "2022-04-17"
        }
      },
      {
        "_index": "blogs",
        "_id": "2",
        "_score": 11.150461,
        "_source": {
          "name": "Get started with Easysearch 2.7",
          "views": 1400,
          "likes": 100,
          "comments": 20,
          "date_posted": "2022-05-02"
        }
      }
    ]
  }
}
```


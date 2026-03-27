---
title: "Script Score 查询"
date: 0001-01-01
summary: "Script Score 查询 #  使用 script_score 查询通过脚本自定义分数计算。对于昂贵的评分函数，您可以使用 script_score 查询仅计算已过滤的返回文档的分数。
相关指南（先读这些） #    相关性与打分策略  Query DSL 基础  参考样例 #  例如，以下请求创建一个包含一个文档的索引：
PUT testindex1/_doc/1 { &#34;name&#34;: &#34;John Doe&#34;, &#34;multiplier&#34;: 0.5 } 您可以使用 match 查询返回所有在 name 字段中包含 John 的文档：
GET testindex1/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;name&#34;: &#34;John&#34; } } } 在返回内容中，文档 1 的得分为 0.2876821 ：
{ &#34;took&#34;: 7, &#34;timed_out&#34;: false, &#34;_shards&#34;: { &#34;total&#34;: 1, &#34;successful&#34;: 1, &#34;skipped&#34;: 0, &#34;failed&#34;: 0 }, &#34;hits&#34;: { &#34;total&#34;: { &#34;value&#34;: 1, &#34;relation&#34;: &#34;eq&#34; }, &#34;max_score&#34;: 0."
---


# Script Score 查询

使用 `script_score` 查询通过脚本自定义分数计算。对于昂贵的评分函数，您可以使用 `script_score` 查询仅计算已过滤的返回文档的分数。

## 相关指南（先读这些）

- [相关性与打分策略]({{< relref "/docs/features/fulltext-search/relevance/_index.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

例如，以下请求创建一个包含一个文档的索引：

```
PUT testindex1/_doc/1
{
  "name": "John Doe",
  "multiplier": 0.5
}
```

您可以使用 `match` 查询返回所有在 `name` 字段中包含 `John` 的文档：

```
GET testindex1/_search
{
  "query": {
    "match": {
      "name": "John"
    }
  }
}
```

在返回内容中，文档 1 的得分为 0.2876821 ：

```
{
  "took": 7,
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
        "_index": "testindex1",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "name": "John Doe",
          "multiplier": 0.5
        }
      }
    ]
  }
}
```

现在我们通过使用一个脚本改变文档得分，该脚本将得分计算为 `_score` 字段的值乘以 `multiplier` 字段的值。在以下查询中，你可以通过 `_score` 变量访问文档的当前相关性得分，并将 `multiplier` 值作为 `doc['multiplier'].value` ：

```
GET testindex1/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
            "name": "John"
        }
      },
      "script": {
        "source": "_score * doc['multiplier'].value"
      }
    }
  }
}
```

文档 1 的得分是原始得分的一半：

```
{
  "took": 8,
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
    "max_score": 0.14384104,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 0.14384104,
        "_source": {
          "name": "John Doe",
          "multiplier": 0.5
        }
      }
    ]
  }
}
```

## 参数说明

`script_score` 查询支持以下顶级参数。

| 参数        | 数据类型 | 描述                                                                                               |
| ----------- | -------- | -------------------------------------------------------------------------------------------------- |
| `query`     | Object   | 用于搜索的查询。必填。                                                                             |
| `script`    | Object   | 用于计算 `query` 返回的文档得分的脚本。必填。                                                      |
| `min_score` | Float    | 排除得分低于 min_score 的文档从结果中。可选。                                                      |
| `boost`     | Float    | 通过给定的倍数提升文档的得分。小于 1.0 的值会降低相关性，大于 1.0 的值会增加相关性。默认值为 1.0。 |

> script_score 查询计算的相关性得分不能为负。

## 使用内置函数自定义评分计算

要自定义评分计算，您可以使用其中一个内置的 Painless 函数。对于每个函数，Easysearch 都提供一个或多个可在脚本评分上下文中访问的 Painless 方法。您可以直接调用以下部分列出的 Painless 方法，而无需使用类名或实例名限定符。

### 饱和度

饱和度函数计算饱和度，其中 `value` 是字段值， `pivot` 被选择以便当 `value` 大于 `pivot` 时评分大于 0.5，当 `value` 小于 `pivot` 时评分小于 0.5。评分在(0, 1)范围内。要应用饱和度函数，请调用以下 Painless 方法：

```
double saturation(double <field-value>, double <pivot>)

```

以下示例查询在 `articles` 索引中搜索文本 `neural search` 。它结合了原始文档相关性得分与 `article_rank` 值，该值首先通过饱和函数进行转换：

```
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "article_name": "neural search" }
      },
      "script" : {
        "source" : "_score + saturation(doc['article_rank'].value, 11)"
      }
    }
  }
}

```

### Sigmoid 函数

与饱和函数类似，sigmoid 函数计算得分为 `score = value^exp/ (value^exp + pivot^exp)` ，其中 `value` 是字段值， `exp` 是指数缩放因子， `pivot` 被选择为当 `value` 大于 `pivot` 时得分为大于 0.5，当 `value` 小于 `pivot` 时得分为小于 0.5。要应用 sigmoid 函数，请调用以下 Painless 方法：

```
double sigmoid(double <field-value>, double <pivot>, double <exp>)

```

以下示例查询在 `articles` 索引中搜索文本 `neural search` 。它结合了原始文档相关性得分与 `article_rank` 值，该值首先通过 sigmoid 函数进行转换：

```
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "article_name": "neural search" }
      },
      "script" : {
        "source" : "_score + sigmoid(doc['article_rank'].value, 11, 2)"
      }
    }
  }
}
```

### 随机得分

随机分数函数在[0, 1)范围内生成均匀分布的随机分数。要应用随机分数函数，请调用以下 Painless 方法之一：

- `double randomScore(int <seed>)` : 使用内部 Lucene 文档 ID 作为种子值。
- `double randomScore(int <seed>, String <field-name>)`

以下查询使用 `random_score` 函数，并带有 `seed` 和 `field` ：

```
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "article_name": "neural search" }
      },
      "script" : {
          "source" : "randomScore(20, '_seq_no')"
      }
    }
  }
}
```

### 衰减函数

使用衰减函数，可以根据邻近度或时效性对结果进行评分。您可以使用指数、高斯或线性衰减曲线来计算分数。要应用衰减函数，根据字段类型调用以下 Painless 方法之一：

- 数值字段：
  - `double decayNumericGauss(double <origin>, double <scale>, double <offset>, double <decay>, double <field-value>)`
  - `double decayNumericExp(double <origin>, double <scale>, double <offset>, double <decay>, double <field-value>)`
  - `double decayNumericLinear(double <origin>, double <scale>, double <offset>, double <decay>, double <field-value>)`
- 地理点字段：
  - `double decayGeoGauss(String <origin>, String <scale>, String <offset>, double <decay>, GeoPoint <field-value>)`
  - `double decayGeoExp(String <origin>, String <scale>, String <offset>, double <decay>, GeoPoint <field-value>)`
  - `double decayGeoLinear(String <origin>, String <scale>, String <offset>, double <decay>, GeoPoint <field-value>)`
- 日期字段：
  - `double decayDateGauss(String <origin>, String <scale>, String <offset>, double <decay>, JodaCompatibleZonedDateTime <field-value>)`
  - `double decayDateExp(String <origin>, String <scale>, String <offset>, double <decay>, JodaCompatibleZonedDateTime <field-value>`
  - `double decayDateLinear(String <origin>, String <scale>, String <offset>, double <decay>, JodaCompatibleZonedDateTime <field-value>)`

#### 示例：数值字段

以下查询在数值字段上使用了指数衰减函数：

```
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
          "article_name": "neural search"
        }
      },
      "script": {
        "source": "decayNumericExp(params.origin, params.scale, params.offset, params.decay, doc['article_rank'].value)",
        "params": {
          "origin": 50,
          "scale": 20,
          "offset": 30,
          "decay": 0.5
        }
      }
    }
  }
}
```

#### 示例：地理点字段

以下查询在地理点字段上使用了高斯衰减函数：

```
GET hotels/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
          "name": "hotel"
        }
      },
      "script": {
        "source": "decayGeoGauss(params.origin, params.scale, params.offset, params.decay, doc['location'].value)",
        "params": {
          "origin": "40.71,74.00",
          "scale":  "300ft",
          "offset": "200ft",
          "decay": 0.25
        }
      }
    }
  }
}
```

#### 示例：日期字段

以下查询在日期字段上使用了线性衰减函数：

```
GET blogs/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
          "name": "easysearch"
        }
      },
      "script": {
        "source": "decayDateLinear(params.origin, params.scale, params.offset, params.decay, doc['date_posted'].value)",
        "params": {
          "origin":  "2022-04-24",
          "scale":  "6d",
          "offset": "1d",
          "decay": 0.25
        }
      }
    }
  }
}
```

### 词频函数

词频函数在评分脚本源中暴露词级统计信息。您可以使用这些统计信息来实现自定义信息检索和排序算法，例如查询时的基于流行度的乘法或加法评分提升。要应用词频函数，请调用以下 Painless 方法之一：

- `int termFreq(String <field-name>, String <term>)` ：检索特定词在字段中的词频。
- `long totalTermFreq(String <field-name>, String <term>)` ：检索特定词在字段中的总词频。
- `long sumTotalTermFreq(String <field-name>)` : 获取字段中总词频的求和。

以下查询将 `fields` 列表中每个字段的词频总和乘以 `multiplier` 值作为分数计算：

```
GET /demo_index_v1/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_all": {}
      },
      "script_score": {
        "script": {
          "source": """
            for (int x = 0; x < params.fields.length; x++) {
              String field = params.fields[x];
              if (field != null) {
                return params.multiplier * totalTermFreq(field, params.term);
              }
            }
            return params.default_value;

          """,
          "params": {
              "fields": ["title", "description"],
              "term": "ai",
              "multiplier": 2,
              "default_value": 1
          }
        }
      }
    }
  }
}
```

> 如果 search.allow_expensive_queries 设置为 false ，则不会执行 script_score 查询。


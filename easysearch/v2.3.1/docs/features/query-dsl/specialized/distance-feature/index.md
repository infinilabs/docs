---
title: "Distance Feature 查询"
date: 0001-01-01
summary: "Distance Feature 查询 #  使用 distance_feature 查询来提升与特定日期或地理位置更近的文档的相关性。这可以帮助你在搜索结果中优先显示更近期的或附近的内容。例如，你可以为近期生产的产品分配更高的权重，或提升最接近用户指定位置的项目。
你可以将此查询应用于包含日期或位置数据的字段。它通常用于 bool 查询的 should 子句中，以改进相关性评分而不过滤掉结果。
相关指南（先读这些） #    相关性与打分策略  地理位置搜索  Query DSL 基础  配置索引 #  在使用 distance_feature 查询之前，请确保您的索引至少包含以下字段类型之一：date,date_nanos,geo_point
在此示例中，您将配置 opening_date 和 coordinates 字段，用于运行距离特征查询：
PUT /stores { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;opening_date&#34;: { &#34;type&#34;: &#34;date&#34; }, &#34;coordinates&#34;: { &#34;type&#34;: &#34;geo_point&#34; } } } } 向索引中添加示例文档：
PUT /stores/_doc/1 { &#34;store_name&#34;: &#34;Green Market&#34;, &#34;opening_date&#34;: &#34;2025-03-10&#34;, &#34;coordinates&#34;: [74.00, 40.70] } PUT /stores/_doc/2 { &#34;store_name&#34;: &#34;Fresh Foods&#34;, &#34;opening_date&#34;: &#34;2025-04-01&#34;, &#34;coordinates&#34;: [73."
---


# Distance Feature 查询

使用 `distance_feature` 查询来提升与特定日期或地理位置更近的文档的相关性。这可以帮助你在搜索结果中优先显示更近期的或附近的内容。例如，你可以为近期生产的产品分配更高的权重，或提升最接近用户指定位置的项目。

你可以将此查询应用于包含日期或位置数据的字段。它通常用于 `bool` 查询的 `should` 子句中，以改进相关性评分而不过滤掉结果。

## 相关指南（先读这些）

- [相关性与打分策略]({{< relref "/docs/features/fulltext-search/relevance/_index.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 配置索引

在使用 `distance_feature` 查询之前，请确保您的索引至少包含以下字段类型之一：`date`,`date_nanos`,`geo_point`

在此示例中，您将配置 `opening_date` 和 `coordinates` 字段，用于运行距离特征查询：

```
PUT /stores
{
  "mappings": {
    "properties": {
      "opening_date": {
        "type": "date"
      },
      "coordinates": {
        "type": "geo_point"
      }
    }
  }
}
```

向索引中添加示例文档：

```
PUT /stores/_doc/1
{
  "store_name": "Green Market",
  "opening_date": "2025-03-10",
  "coordinates": [74.00, 40.70]
}

PUT /stores/_doc/2
{
  "store_name": "Fresh Foods",
  "opening_date": "2025-04-01",
  "coordinates": [73.98, 40.75]
}

PUT /stores/_doc/3
{
  "store_name": "City Organics",
  "opening_date": "2021-04-20",
  "coordinates": [74.02, 40.68]
}
```

## 示例：根据时效性提升分数

以下查询搜索匹配 `store_name` 的文档，并提升最近打开的商店的分数：

```
GET /stores/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "store_name": "market"
        }
      },
      "should": {
        "distance_feature": {
          "field": "opening_date",
          "origin": "2025-04-07",
          "pivot": "10d"
        }
      }
    }
  }
}
```

结果包含匹配的文档：

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
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.2372394,
    "hits": [
      {
        "_index": "stores",
        "_id": "1",
        "_score": 1.2372394,
        "_source": {
          "store_name": "Green Market",
          "opening_date": "2025-03-10",
          "coordinates": [
            74,
            40.7
          ]
        }
      }
    ]
  }
}

```

## 根据地理邻近度提升分数

以下查询搜索匹配 `store_name` 的文档，并提升更接近给定原点的结果：

```
GET /stores/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "store_name": "market"
        }
      },
      "should": {
        "distance_feature": {
          "field": "coordinates",
          "origin": [74.00, 40.71],
          "pivot": "500m"
        }
      }
    }
  }
}
```

结果包含匹配的文档：

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
    "max_score": 1.2910118,
    "hits": [
      {
        "_index": "stores",
        "_id": "1",
        "_score": 1.2910118,
        "_source": {
          "store_name": "Green Market",
          "opening_date": "2025-03-10",
          "coordinates": [
            74,
            40.7
          ]
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `distance_feature` 查询支持的所有顶层参数。

| 参数     | 必需/可选 | 描述                                                                                                                                                              |
| -------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`  | 必需      | 计算距离所使用的字段的名称。必须是 date 、 date_nanos 或 geo_point 字段，具有 index: true （默认值）和 doc_values: true （默认值）。                              |
| `origin` | 必需      | 用于计算距离的起点。对于 date 字段，使用日期或日期数学表达式（例如， now-1h ）；对于 geo_point 字段，使用地理点。                                                 |
| `pivot`  | 必需      | 在距离 origin 处，匹配文档的分数获得 boost 值的一半。对于日期字段，使用时间单位（例如， 10d ）；对于地理字段，使用距离单位（例如， 1km ）。更多信息，请参阅单位。 |
| `boost`  | 可选      | 匹配文档的相关性分数的乘数。必须是非负浮点数。默认值是 1.0 。                                                                                                     |

## 跳过非竞争性命中

与 `function_score` 查询等其他修改得分的查询不同， `distance_feature` 查询在禁用总命中跟踪（ `track_total_hits` ）时经过优化，能够高效地跳过非竞争性命中。


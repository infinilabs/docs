---
title: "复合聚合（Composite）"
date: 0001-01-01
summary: "复合聚合 #  composite 聚合基于一个或多个文档字段或源创建分组。composite 聚合为每个单独源值的组合创建一个分组。默认情况下，如果一个或多个字段中缺少值，则这些组合会从结果中省略。
相关指南（先读这些） #    聚合基础  聚合场景实践  每个源有四种类型的聚合之一：
 terms 类型按唯一（通常是 String ）值分组。 histogram 类型按指定宽度数值分组。 date_histogram 类型按指定宽度的日期或时间范围分组。 geotile_grid 类型按指定分辨率将地理点分组到网格中。  composite 聚合通过将其源键组合到分组中来工作。生成的分组是有序的，跨源(Across)和源内部(Within)都是：
 Across：分组按聚合请求中源的顺序嵌套。 Within:每个源中值的顺序决定了该源的分组顺序。排序方式根据源类型适当选择，可以是字母顺序、数字顺序、日期时间顺序或地理切片顺序。  考虑一下来自马拉松参与者索引的这些字段：
{... &#34;city&#34;: &#34;Albuquerque&#34;, &#34;place&#34;: &#34;Bronze&#34; ...} {... &#34;city&#34;: &#34;Boston&#34;, ...} {... &#34;city&#34;: &#34;Chicago&#34;, &#34;place&#34;: &#34;Bronze&#34; ...} {... &#34;city&#34;: &#34;Albuquerque&#34;, &#34;place&#34;: &#34;Gold&#34; ...} {... &#34;city&#34;: &#34;Chicago&#34;, &#34;place&#34;: &#34;Silver&#34; ...} {... &#34;city&#34;: &#34;Boston&#34;, &#34;place&#34;: &#34;Bronze&#34; ...} {... &#34;city&#34;: &#34;Chicago&#34;, &#34;place&#34;: &#34;Gold&#34; ."
---


# 复合聚合

`composite` 聚合基于一个或多个文档字段或源创建分组。`composite` 聚合为每个单独源值的组合创建一个分组。默认情况下，如果一个或多个字段中缺少值，则这些组合会从结果中省略。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

每个源有四种类型的聚合之一：

- `terms` 类型按唯一（通常是 `String` ）值分组。
- `histogram` 类型按指定宽度数值分组。
- `date_histogram` 类型按指定宽度的日期或时间范围分组。
- `geotile_grid` 类型按指定分辨率将地理点分组到网格中。

`composite` 聚合通过将其源键组合到分组中来工作。生成的分组是有序的，跨源(`Across`)和源内部(`Within`)都是：

- `Across`：分组按聚合请求中源的顺序嵌套。
- `Within`:每个源中值的顺序决定了该源的分组顺序。排序方式根据源类型适当选择，可以是字母顺序、数字顺序、日期时间顺序或地理切片顺序。

考虑一下来自马拉松参与者索引的这些字段：

```
{... "city": "Albuquerque", "place": "Bronze" ...}
{... "city": "Boston",  ...}
{... "city": "Chicago", "place": "Bronze" ...}
{... "city": "Albuquerque", "place": "Gold" ...}
{... "city": "Chicago", "place": "Silver" ...}
{... "city": "Boston", "place": "Bronze" ...}
{... "city": "Chicago", "place": "Gold" ...}
```

假设请求指定源如下：

```
    ...
    "sources": [
        { "marathon_city": { "terms": { "field": "city" }}},
        { "participant_medal": { "terms": { "field": "place" }}}
    ],
    ...
```

> 你必须为每个源分配一个唯一的键名。

生成的 `composite` 包含以下分组，按顺序排列：

```
{ "city": "Albuquerque", "place": "Bronze" }
{ "city": "Albuquerque", "place": "Gold" }
{ "city": "Boston", "place": "Bronze" }
{ "city": "Boston", "place": "Silver" }
{ "city": "Chicago", "place": "Bronze" }
{ "city": "Chicago", "place": "Gold" }
{ "city": "Chicago", "place": "Silver" }
```

请注意， `city` 和 `place` 字段都是按字母顺序排列的。

## 参数说明

`composite` 聚合采用以下参数。

| 参数             | 必需/可选 | 数据类型 | 描述                                                                                                                                                       |
| ---------------- | --------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sources`        | 必填      | Array    | 源对象的数组。有效类型为 `terms` 、 `histogram` 、 `date_histogram` 和 `geotile_grid` 。                                                                   |
| `size`           | 可选      | Numeric  | 在结果中返回的 `composite` 分组的数量。默认值为 10 。参见 分页复合结果。                                                                                   |
| `after`          | 可选      | String   | 一个键，用于指定从何处继续显示分页的 `composite` 分组。参见 分页复合结果。                                                                                 |
| `order`          | 可选      | String   | 对于每个数据源，是否按升序或降序排列值。有效值为 `asc` 和 `desc` 。默认值为 `asc` 。                                                                       |
| `missing_bucket` | 可选      | Boolean  | 对于每个数据源，是否包含缺失值文档。默认值为 `false` 。如果设置为 `true` ，Easysearch 会包含这些文档，并将 null 作为字段的键。空值在升序排列中排在最前面。 |

> 有关聚合特定参数，请参阅相应的聚合文档。

## Terms

使用 `terms` 聚合来聚合字符串或布尔数据。更多信息，请参阅 Terms 聚合。

> 您可以使用 terms 源来为任何类型的数据创建复合分组。但是，由于 terms 源为每个唯一值创建分组，因此您通常使用 histogram 源来代替数值数据。

以下示例请求返回数据中星期几和客户性别的第一个 4 复合分组：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "composite_buckets": {
      "composite": {
        "sources": [
          { "day": { "terms": { "field": "day_of_week" }}},
          { "gender": { "terms": { "field": "customer_gender" }}}
        ],
        "size": 4
      }
    }
  }
}

```

由于此示例的数据集包含每个分组的有效数据，因此聚合会为性别和星期几的每一组合生成一个分组，最终产生 14 个总分组。

由于请求指定了 `size` 个 4 ，响应包含前四个复合分组。由于源是 `terms` ，分组在跨源和源内部都是按升序字母顺序排列的：

```
{
  "took": 51,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "composite_buckets": {
      "after_key": {
        "day": "Monday",
        "gender": "MALE"
      },
      "buckets": [
        {
          "key": {
            "day": "Friday",
            "gender": "FEMALE"
          },
          "doc_count": 399
        },
        {
          "key": {
            "day": "Friday",
            "gender": "MALE"
          },
          "doc_count": 371
        },
        {
          "key": {
            "day": "Monday",
            "gender": "FEMALE"
          },
          "doc_count": 320
        },
        {
          "key": {
            "day": "Monday",
            "gender": "MALE"
          },
          "doc_count": 259
        }
      ]
    }
  }
}
```

您可以使用返回的 `after_key` 来查看更多结果。请参阅下一节的示例。

## 直方图

使用 `histogram` 个数据源创建数值数据的组合聚合。更多信息，请参阅直方图聚合。

对于 `histogram` 个数据源，每个 `composite` 分组键中使用的名称是该键的直方图间隔中的最低值。每个源直方图间隔包含 `[lower_bound, lower_bound + interval)` 范围内的值。第一个间隔的名称是源字段中的最低值（对于升序值源）。

以下示例请求根据分组宽度分别为 1 和 50 ，返回数据中数量和基本单位价格的前 6 个组合分组：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "composite_buckets": {
      "composite": {
        "sources": [
          { "quantity": { "histogram": { "field": "products.quantity", "interval": 1 }}},
          { "unit_price": { "histogram": { "field": "products.base_unit_price", "interval": 50 }}}
        ],
        "size": 6
      }
    }
  }
}
```

聚合返回两个 `histogram` 源的第一个 6 分组键和文档计数。与 `terms` 示例一样，分组在源字段之间和内部按顺序排列。然而在此情况下，顺序是数值的，基于每个直方图宽度的包含下界：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "composite_buckets": {
      "after_key": {
        "quantity": 2,
        "unit_price": 150
      },
      "buckets": [
        {
          "key": {
            "quantity": 1,
            "unit_price": 0
          },
          "doc_count": 17691
        },
        {
          "key": {
            "quantity": 1,
            "unit_price": 50
          },
          "doc_count": 5014
        },
        {
          "key": {
            "quantity": 1,
            "unit_price": 100
          },
          "doc_count": 482
        },
        {
          "key": {
            "quantity": 1,
            "unit_price": 150
          },
          "doc_count": 148
        },
        {
          "key": {
            "quantity": 1,
            "unit_price": 200
          },
          "doc_count": 32
        },
        {
          "key": {
            "quantity": 2,
            "unit_price": 150
          },
          "doc_count": 4
        }
      ]
    }
  }
}
```

每个字段的分组键是字段区间的下界。例如，第一个 `composite` 分组的 `unit_price` 键是 0 。

要检索下一个 6 分组，请按如下方式将响应中的 `after_key` 对象提供给 `after` 参数：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "composite_buckets": {
      "composite": {
        "sources": [
          { "quantity": { "histogram": { "field": "products.quantity", "interval": 1 }}},
          { "unit_price": { "histogram": { "field": "products.base_unit_price", "interval": 50 }}}
        ],
        "size": 6,
        "after": {
            "quantity": 2,
            "unit_price": 150
        }
      }
    }
  }
}

```

仅剩两个分组：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "composite_buckets": {
      "after_key": {
        "quantity": 2,
        "unit_price": 500
      },
      "buckets": [
        {
          "key": {
            "quantity": 2,
            "unit_price": 200
          },
          "doc_count": 8
        },
        {
          "key": {
            "quantity": 2,
            "unit_price": 500
          },
          "doc_count": 4
        }
      ]
    }
  }
}

```

## 日期直方图

要创建日期范围的组合聚合，请使用 `date_histogram` 聚合。有关详细信息，请参阅日期直方图聚合。

Easysearch 将日期（包括 `date_interval` 分组键）表示为 `long` 表示自 Unix 时间以来的毫秒数的整数。您可以使用 `format` 参数格式化日期输出。这不会改变键的顺序。

Easysearch 以 UTC 存储日期和时间。您可以使用 `time_zone` 参数以不同的时区显示输出结果。

以下示例请求返回数据中，每个售出产品创建年份的第一组 4 复合分组，以及售出日期，基于分组宽度分别为 1 年和 1 天：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "composite_buckets": {
      "composite": {
        "sources": [
          { "product_creation_date": { "date_histogram": { "field": "products.created_on", "calendar_interval": "1y", "format": "yyyy" }}},
          { "order_date": { "date_histogram": { "field": "order_date", "calendar_interval": "1d", "format": "yyyy-MM-dd" }}}
        ],
        "size": 4
      }
    }
  }
}

```

聚合返回格式化的基于日期的分组键和计数。对于 `date_interval` 复合聚合，字段排序按日期：

```
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "composite_buckets": {
      "after_key": {
        "product_creation_date": "2016",
        "order_date": "2025-02-23"
      },
      "buckets": [
        {
          "key": {
            "product_creation_date": "2016",
            "order_date": "2025-02-20"
          },
          "doc_count": 146
        },
        {
          "key": {
            "product_creation_date": "2016",
            "order_date": "2025-02-21"
          },
          "doc_count": 153
        },
        {
          "key": {
            "product_creation_date": "2016",
            "order_date": "2025-02-22"
          },
          "doc_count": 143
        },
        {
          "key": {
            "product_creation_date": "2016",
            "order_date": "2025-02-23"
          },
          "doc_count": 140
        }
      ]
    }
  }
}

```

## 地理网格

使用 `geotile_grid` 源将 `geo_point` 值聚合到代表地图瓦片的分组中。与其他复合聚合源一样，默认情况下，结果仅包括包含数据的分组。有关更多信息，请参阅 `Geotile` 网格聚合。

每个单元格对应一个地图瓦片。单元格标签使用 `{zoom}/{x}/{y}` 格式。

以下示例请求返回以 8 精度包含 `geoip.location` 字段中的位置的前 6 个瓦片：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "composite_buckets": {
      "composite": {
        "sources": [
          { "tile": { "geotile_grid": { "field": "geoip.location", "precision": 8 } } }
        ],
        "size": 6
      }
    }
  }
}

```

聚合返回指定的 `geo_tiles` 和点计数：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "composite_buckets": {
      "after_key": {
        "tile": "8/122/104"
      },
      "buckets": [
        {
          "key": {
            "tile": "8/43/102"
          },
          "doc_count": 310
        },
        {
          "key": {
            "tile": "8/75/96"
          },
          "doc_count": 896
        },
        {
          "key": {
            "tile": "8/75/124"
          },
          "doc_count": 178
        },
        {
          "key": {
            "tile": "8/122/104"
          },
          "doc_count": 408
        }
      ]
    }
  }
}

```

## 组合源

您可以组合两个或多个任何不同类型的来源。

以下示例请求返回由三种不同来源类型组成的分组：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "composite_buckets": {
      "composite": {
        "sources": [
          { "order_date": { "date_histogram": { "field": "order_date", "calendar_interval": "1M", "format": "yyyy-MM" }}},
          { "gender": { "terms": { "field": "customer_gender" }}},
          { "unit_price": { "histogram": { "field": "products.base_unit_price", "interval": 200 }}}
        ],
        "size": 10
      }
    }
  }
}
```

聚合返回混合类型的 `composite` 分组和文档计数：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "composite_buckets": {
      "after_key": {
        "order_date": "2025-03",
        "gender": "MALE",
        "unit_price": 200
      },
      "buckets": [
        {
          "key": {
            "order_date": "2025-02",
            "gender": "FEMALE",
            "unit_price": 0
          },
          "doc_count": 1517
        },
        {
          "key": {
            "order_date": "2025-02",
            "gender": "MALE",
            "unit_price": 0
          },
          "doc_count": 1369
        },
        {
          "key": {
            "order_date": "2025-02",
            "gender": "MALE",
            "unit_price": 200
          },
          "doc_count": 6
        },
        {
          "key": {
            "order_date": "2025-02",
            "gender": "MALE",
            "unit_price": 400
          },
          "doc_count": 1
        },
        {
          "key": {
            "order_date": "2025-03",
            "gender": "FEMALE",
            "unit_price": 0
          },
          "doc_count": 3656
        },
        {
          "key": {
            "order_date": "2025-03",
            "gender": "FEMALE",
            "unit_price": 200
          },
          "doc_count": 1
        },
        {
          "key": {
            "order_date": "2025-03",
            "gender": "MALE",
            "unit_price": 0
          },
          "doc_count": 3530
        },
        {
          "key": {
            "order_date": "2025-03",
            "gender": "MALE",
            "unit_price": 200
          },
          "doc_count": 7
        }
      ]
    }
  }
}
```

## 子聚合

组合聚合在结合子聚合时最为有用，子聚合可以揭示 `composite` 分组中文档的信息。

以下示例请求比较了数据中每周每天按性别划分的平均支出：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "composite_buckets": {
      "composite": {
        "sources": [
          { "weekday": { "terms": { "field": "day_of_week" }}},
          { "gender": { "terms": { "field": "customer_gender" }}}
        ],
        "size": 6
      },
      "aggs": {
        "avg_spend": {
          "avg": { "field": "taxful_total_price" }
        }
      }
    }
  }
}
```

该聚合返回前 6 个分组的平均 `taxful_total_price` 值：

```
{
  "took": 30,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "composite_buckets": {
      "after_key": {
        "weekday": "Saturday",
        "gender": "MALE"
      },
      "buckets": [
        {
          "key": {
            "weekday": "Friday",
            "gender": "FEMALE"
          },
          "doc_count": 399,
          "avg_spend": {
            "value": 71.7733395989975
          }
        },
        {
          "key": {
            "weekday": "Friday",
            "gender": "MALE"
          },
          "doc_count": 371,
          "avg_spend": {
            "value": 79.72514108827494
          }
        },
        {
          "key": {
            "weekday": "Monday",
            "gender": "FEMALE"
          },
          "doc_count": 320,
          "avg_spend": {
            "value": 72.1588623046875
          }
        },
        {
          "key": {
            "weekday": "Monday",
            "gender": "MALE"
          },
          "doc_count": 259,
          "avg_spend": {
            "value": 86.1754946911197
          }
        },
        {
          "key": {
            "weekday": "Saturday",
            "gender": "FEMALE"
          },
          "doc_count": 365,
          "avg_spend": {
            "value": 73.53236301369863
          }
        },
        {
          "key": {
            "weekday": "Saturday",
            "gender": "MALE"
          },
          "doc_count": 371,
          "avg_spend": {
            "value": 72.78092360175202
          }
        }
      ]
    }
  }
}

```

## 分页复合结果

如果请求导致超过 `size` 个分组，则返回 `size` 个分组。在这种情况下，结果包含一个 `after_key` 对象，其中包含列表中下一个分组的键。要检索请求的下一个 `size` 个分组，请再次发送请求，并在 `after` 参数中提供 `after_key` 。例如，请参阅 `Histogram` 中的请求。

> 始终使用 after_key 继续分页响应，而不是复制最后一个分组。两者有时是不同的。

## 使用索引排序提升性能

为了加快在大数据集上的复合聚合速度，你可以使用与聚合源相同的字段和顺序来对索引进行排序。当 `index.sort.field` 和 `index.sort.order` 与复合聚合中使用的源字段和顺序相匹配时，Easysearch 可以更高效地返回结果，并减少内存使用。虽然索引排序在索引过程中会带来轻微的开销，但复合聚合的查询性能提升非常显著。

以下示例请求为 `my-sorted-index` 索引中的每个字段设置了排序字段和排序顺序：

```
PUT /my-sorted-index
{
  "settings": {
    "index": {
      "sort.field": ["customer_id", "timestamp"],
      "sort.order": ["asc", "desc"]
    }
  },
  "mappings": {
    "properties": {
      "customer_id": {
        "type": "keyword"
      },
      "timestamp": {
        "type": "date"
      },
      "price": {
        "type": "double"
      }
    }
  }
}

```

以下请求在 `my-sorted-index` 索引上创建复合聚合。由于该索引按 `customer_id` 升序排序，按 `timestamp` 降序排序，并且聚合源与该排序顺序匹配，因此此查询运行更快，且内存压力减小：

```
GET /my-sorted-index/_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "size": 1000,
        "sources": [
          { "customer": { "terms": { "field": "customer_id", "order": "asc" } } },
          { "time": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d", "order": "desc" } } }
        ]
      }
    }
  }
}
```


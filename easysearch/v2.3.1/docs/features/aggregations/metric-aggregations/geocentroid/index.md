---
title: "地理中心点聚合（Geo Centroid）"
date: 0001-01-01
summary: "地理中心点聚合 #  geo_centroid 聚合计算一组 geo_point 值的地理中心或焦点。它将中心位置作为纬度-经度对返回。
相关指南（先读这些） #    聚合基础  地理位置搜索  Geo 场景实践  参数说明 #  geo_centroid 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 包含计算中心的地理点的字段的名称。    参考样例 #  以下示例返回数据中每个订单的 geo_centroid 的 geoip.location 。每个 geoip.location 都是一个地理点：
GET /sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;centroid&#34;: { &#34;geo_centroid&#34;: { &#34;field&#34;: &#34;geoip.location&#34; } } } } 返回内容 #  返回内容包括一个 centroid 对象，该对象具有 lat 和 lon 属性，表示所有索引数据点的中心位置："
---


# 地理中心点聚合

`geo_centroid` 聚合计算一组 `geo_point` 值的地理中心或焦点。它将中心位置作为纬度-经度对返回。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [Geo 场景实践]({{< relref "/docs/features/geo-search/geo-recipes" >}})

## 参数说明

`geo_centroid` 聚合采用以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| ---------------- | --------- | -------- | ------------------------------------------------------ |
| `field` | 必需 | String | 包含计算中心的地理点的字段的名称。 |

## 参考样例

以下示例返回数据中每个订单的 `geo_centroid` 的 `geoip.location` 。每个 `geoip.location` 都是一个地理点：

```
GET /sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "centroid": {
      "geo_centroid": {
        "field": "geoip.location"
      }
    }
  }
}
```

## 返回内容

返回内容包括一个 `centroid` 对象，该对象具有 lat 和 lon 属性，表示所有索引数据点的中心位置：

```
{
  "took": 35,
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
    "centroid": {
      "location": {
        "lat": 35.54990372113027,
        "lon": -9.079764742533712
      },
      "count": 4675
    }
  }
}

```

中心位置位于摩洛哥以北约的大西洋中。鉴于数据库中订单的广泛地理分布，这并不是很有意义。

## 嵌套在其他聚合下使用

您可以将 `geo_centroid` 聚合嵌套在 bucket 聚合中，以计算数据子集的中心。

### 示例：嵌套在分组聚合下

您可以在字符串字段的 `terms` 分组下嵌套 `geo_centroid` 聚合。

要找到每个大洲的订单的 `geoip` 中心位置，请在 `geoip.continent_name` 字段内对中心进行子聚合：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "continents": {
      "terms": {
        "field": "geoip.continent_name"
      },
      "aggs": {
        "centroid": {
          "geo_centroid": {
            "field": "geoip.location"
          }
        }
      }
    }
  }
}

```

这将返回每个大陆分组的中心位置：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "continents": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Asia",
          "doc_count": 1220,
          "centroid": {
            "location": {
              "lat": 28.023606536509163,
              "lon": 47.83377046025068
            },
            "count": 1220
          }
        },
        {
          "key": "North America",
          "doc_count": 1206,
          "centroid": {
            "location": {
              "lat": 39.06542286878007,
              "lon": -85.36152573149485
            },
            "count": 1206
          }
        },
        {
          "key": "Europe",
          "doc_count": 1172,
          "centroid": {
            "location": {
              "lat": 48.125767892293325,
              "lon": 2.7529009746915243
            },
            "count": 1172
          }
        },
        {
          "key": "Africa",
          "doc_count": 899,
          "centroid": {
            "location": {
              "lat": 30.780756367941297,
              "lon": 13.464182392125318
            },
            "count": 899
          }
        },
        {
          "key": "South America",
          "doc_count": 178,
          "centroid": {
            "location": {
              "lat": 4.599999985657632,
              "lon": -74.10000007599592
            },
            "count": 178
          }
        }
      ]
    }
  }
}
```


---
title: "Geo Distance 查询"
date: 0001-01-01
description: "圆形距离过滤：参数、距离单位与计算方式。"
summary: "Geo Distance 查询 #  地理距离查询返回 geo_point 字段值位于指定中心点给定距离范围内的文档。如果一个文档包含多个地理坐标点，只要至少有一个点在距离范围内，该文档就匹配。
参考样例 #  创建一个映射，将 point 字段映射为 geo_point ：
PUT testindex1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;point&#34;: { &#34;type&#34;: &#34;geo_point&#34; } } } } 索引一个地理点，指定其纬度和经度：
PUT testindex1/_doc/1 { &#34;point&#34;: { &#34;lat&#34;: 74.00, &#34;lon&#34;: 40.71 } } 搜索距离指定的 point 内包含指定 distance 对象的文档：
GET /testindex1/_search { &#34;query&#34;: { &#34;bool&#34;: { &#34;must&#34;: { &#34;match_all&#34;: {} }, &#34;filter&#34;: { &#34;geo_distance&#34;: { &#34;distance&#34;: &#34;50mi&#34;, &#34;point&#34;: { &#34;lat&#34;: 73.5, &#34;lon&#34;: 40."
---


# Geo Distance 查询

地理距离查询返回 `geo_point` 字段值位于指定中心点给定距离范围内的文档。如果一个文档包含多个地理坐标点，只要至少有一个点在距离范围内，该文档就匹配。

## 参考样例

创建一个映射，将 `point` 字段映射为 `geo_point` ：

```
PUT testindex1
{
  "mappings": {
    "properties": {
      "point": {
        "type": "geo_point"
      }
    }
  }
}
```

索引一个地理点，指定其纬度和经度：

```
PUT testindex1/_doc/1
{
  "point": {
    "lat": 74.00,
    "lon": 40.71
  }
}
```

搜索距离指定的 `point` 内包含指定 `distance` 对象的文档：

```
GET /testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "50mi",
          "point": {
            "lat": 73.5,
            "lon": 40.5
          }
        }
      }
    }
  }
}
```

返回内容包含匹配的文档：

```
{
  "took": 5,
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
        "_index": "testindex1",
        "_id": "1",
        "_score": 1,
        "_source": {
          "point": {
            "lat": 74,
            "lon": 40.71
          }
        }
      }
    ]
  }
}

```

## 参数说明

Geodistance 查询接受以下参数。

| 参数                | 数据类型 | 描述                                                                                                                                                       |
| ------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_name`             | String   | 过滤器名称。可选。                                                                                                                                         |
| `distance`          | String   | 匹配点的距离范围。此距离是以指定点为中心的圆的半径。有关支持的距离单位，请参阅距离单位。必需。                                                             |
| `distance_type`     | String   | 指定如何计算距离。有效值是 arc 或 plane （对于长距离或接近极点的点，速度快但不够准确）。可选。默认是 arc 。                                                |
| `validation_method` | String   | 验证方法。有效值是 IGNORE_MALFORMED （接受坐标无效的地理点）、 COERCE （尝试将坐标转换为有效值）和 STRICT （当坐标无效时返回错误）。可选。默认是 STRICT 。 |
| `ignore_unmapped`   | Boolean  | 指定是否忽略未映射的字段。如果设置为 true ，则查询不返回包含未映射字段的任何文档。如果设置为 false ，则在字段未映射时抛出异常。可选。默认为 false 。       |
| `boost`             | Float    | 查询的相关性得分权重。默认为 `1.0`。                                                                                                                                          |


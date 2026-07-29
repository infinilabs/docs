---
title: "Geo Polygon 查询"
date: 0001-01-01
description: "多边形区域过滤：参数与坐标顺序。"
summary: "Geo Polygon 查询 #  地理多边形查询返回 geo_point 字段值位于指定多边形内的文档。如果一个文档包含多个地理坐标点，只要至少有一个点在多边形内，该文档就匹配。
多边形通过坐标顶点列表指定，不需要闭合（首尾相同不是必须的），但建议按顺时针或逆时针顺序列出。
参考样例 #  创建一个映射，将 point 字段映射为 geo_point ：
PUT /testindex1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;point&#34;: { &#34;type&#34;: &#34;geo_point&#34; } } } } 索引一个地理点，指定其纬度和经度：
PUT testindex1/_doc/1 { &#34;point&#34;: { &#34;lat&#34;: 73.71, &#34;lon&#34;: 41.32 } } 搜索包含指定 geo_polygon 的 point 对象的文档：
GET /testindex1/_search { &#34;query&#34;: { &#34;bool&#34;: { &#34;must&#34;: { &#34;match_all&#34;: {} }, &#34;filter&#34;: { &#34;geo_polygon&#34;: { &#34;point&#34;: { &#34;points&#34;: [ { &#34;lat&#34;: 74."
---


# Geo Polygon 查询

地理多边形查询返回 `geo_point` 字段值位于指定多边形内的文档。如果一个文档包含多个地理坐标点，只要至少有一个点在多边形内，该文档就匹配。

多边形通过坐标顶点列表指定，不需要闭合（首尾相同不是必须的），但建议按顺时针或逆时针顺序列出。

## 参考样例

创建一个映射，将 `point` 字段映射为 `geo_point` ：

```
PUT /testindex1
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
    "lat": 73.71,
    "lon": 41.32
  }
}
```

搜索包含指定 `geo_polygon` 的 `point` 对象的文档：

```
GET /testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_polygon": {
          "point": {
            "points": [
              { "lat": 74.5627, "lon": 41.8645 },
              { "lat": 73.7562, "lon": 42.6526 },
              { "lat": 73.3245, "lon": 41.6189 },
              { "lat": 74.0060, "lon": 40.7128 }
           ]
          }
        }
      }
    }
  }
}
```

前一个请求中指定的地理位置形成一个四边形。匹配的文档位于此四边形内。四边形的顶点坐标以 (latitude, longitude) 格式指定。

返回内容包含匹配的文档：

```
{
  "took": 6,
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
            "lat": 73.71,
            "lon": 41.32
          }
        }
      }
    ]
  }
}
```

在先前的搜索请求中，您指定了多边形的顶点按顺时针顺序：

```
"geo_polygon": {
    "point": {
    "points": [
        { "lat": 74.5627, "lon": 41.8645 },
        { "lat": 73.7562, "lon": 42.6526 },
        { "lat": 73.3245, "lon": 41.6189 },
        { "lat": 74.0060, "lon": 40.7128 }
    ]
    }
}
```

或者，您也可以按逆时针顺序指定顶点：

```
"geo_polygon": {
    "point": {
    "points": [
        { "lat": 74.5627, "lon": 41.8645 },
        { "lat": 74.0060, "lon": 40.7128 },
        { "lat": 73.3245, "lon": 41.6189 },
        { "lat": 73.7562, "lon": 42.6526 }
    ]
    }
}
```

查询结果包含相同的匹配文档。

然而，如果您按照以下顺序指定顶点：

```
"geo_polygon": {
    "point": {
    "points": [
        { "lat": 74.5627, "lon": 41.8645 },
        { "lat": 74.0060, "lon": 40.7128 },
        { "lat": 73.7562, "lon": 42.6526 },
        { "lat": 73.3245, "lon": 41.6189 }
    ]
    }
}
```

则不返回任何结果。

## 参数说明

Geopolygon 查询接受以下参数。

| 参数                | 数据类型 | 描述                                                                                                                                                       |
| ------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_name`             | String   | 过滤器名称。可选。                                                                                                                                         |
| `validation_method` | String   | 验证方法。有效值是 IGNORE_MALFORMED （接受坐标无效的地理点）、 COERCE （尝试将坐标转换为有效值）和 STRICT （当坐标无效时返回错误）。可选。默认是 STRICT 。 |
| `ignore_unmapped`   | Boolean  | 指定是否忽略未映射的字段。如果设置为 true ，则查询不返回包含未映射字段的任何文档。如果设置为 false ，则在字段未映射时抛出异常。可选。默认为 false 。       |
| `boost`             | Float    | 查询的相关性得分权重。默认为 `1.0`。                                                                                                                                          |


---
title: "地理坐标点字段类型（Geo Point）"
date: 0001-01-01
summary: "地理坐标点字段类型（Geo Point） #  geo_point 字段类型包含由纬度（latitude）和经度（longitude）指定的地理点。地理坐标点可以用来计算两个坐标间的距离，判断一个坐标是否在一个区域中，或在聚合中。
代码示例 #  创建一个带有 Geopoint 地理点字段类型的映射：
PUT testindex1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;point&#34;: { &#34;type&#34;: &#34;geo_point&#34; } } } }  地理坐标点不能被动态映射自动检测，必须显式声明字段类型为 geo_point。
 地理点格式 #  Geopoint 地理点可以用以下格式索引：
 包含纬度和经度的对象  PUT testindex1/_doc/1 { &#34;point&#34;: { &#34;lat&#34;: 40.71, &#34;lon&#34;: 74.00 } } 写入包含纬度,经度 的文档  PUT testindex1/_doc/2 { &#34;point&#34;: &#34;40.71,74.00&#34; } geohash 格式的文档  PUT testindex1/_doc/3 { &#34;point&#34;: &#34;txhxegj0uyp3&#34; } [经度, 纬度] 格式的数组  PUT testindex1/_doc/4 { &#34;point&#34;: [74."
---


# 地理坐标点字段类型（Geo Point）

`geo_point` 字段类型包含由纬度（latitude）和经度（longitude）指定的地理点。地理坐标点可以用来计算两个坐标间的距离，判断一个坐标是否在一个区域中，或在聚合中。

## 代码示例

创建一个带有 Geopoint 地理点字段类型的映射：

```json
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

> 地理坐标点不能被动态映射自动检测，必须显式声明字段类型为 `geo_point`。

## 地理点格式

Geopoint 地理点可以用以下格式索引：

1. 包含纬度和经度的对象

```json
PUT testindex1/_doc/1
{
  "point": {
    "lat": 40.71,
    "lon": 74.00
  }
}
```

2. 写入包含`纬度,经度` 的文档

```json
PUT testindex1/_doc/2
{
  "point": "40.71,74.00"
}
```

3. geohash 格式的文档

```json
PUT testindex1/_doc/3
{
  "point": "txhxegj0uyp3"
}
```

4. [`经度`, `纬度`] 格式的数组

```json
PUT testindex1/_doc/4
{
  "point": [74.00, 40.71]
}
```

5. [Well-Known Text](https://docs.ogc.org/is/12-063r5/12-063r5.html) POINT 格式，格式为 "POINT(`经度` `纬度`)"

```json
PUT testindex1/_doc/5
{
  "point": "POINT (74.00 40.71)"
}
```

> **注意坐标顺序**：地理坐标点用字符串形式表示时是**纬度在前，经度在后**（`"latitude,longitude"`），而数组形式表示时是**经度在前，纬度在后**（`[longitude,latitude]`）——顺序刚好相反。这是为了兼容 GeoJSON 格式规范。

## 参数说明

下表列出了 Geopoint 地理点字段类型接受的参数。所有参数都是可选的。

| 参数               | 描述                                                                                                                                                          |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ignore_malformed` | 布尔值，指定是否忽略格式错误的值而不抛出异常。纬度的有效值范围是 [-90, 90]。经度的有效值范围是 [-180, 180]。默认为 `false`。                                  |
| `ignore_z_value`   | 特定于具有三个坐标的点。如果 `ignore_z_value` 为 true，则第三个坐标不会被索引，但仍会存储在 \_source 字段中。如果 `ignore_z_value` 为 `false`，则会抛出异常。 |
| `null_value`       | 用于替代 null 的值。必须与字段类型相同。如果未指定此参数，当字段值为 `null` 时，该字段将被视为缺失。默认为 `null`。                                           |

## 支持的查询

`geo_point` 字段支持以下地理查询：

| 查询类型 | 说明 |
|----------|------|
| [Geo Bounding Box]({{< relref "/docs/features/geo-search/geo-bounding-box.md" >}}) | 矩形区域过滤，最高效的地理过滤器 |
| [Geo Distance]({{< relref "/docs/features/geo-search/geo-distance.md" >}}) | 指定距离内的圆形范围过滤 |
| [Geo Polygon]({{< relref "/docs/features/geo-search/geo-polygon.md" >}}) | 多边形区域过滤 |

> **性能提示**：`geo_bounding_box` 是最高效的地理过滤器。建议先用矩形过滤排除大部分文档，再配合更精确的距离过滤。布尔过滤器 `bool` 会自动优先执行高效过滤。

## 按距离排序

检索结果可以按与指定点的距离排序：

```json
GET /attractions/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left":     { "lat": 40.8, "lon": -74.0 },
            "bottom_right": { "lat": 40.4, "lon": -73.0 }
          }
        }
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location":      { "lat": 40.715, "lon": -73.998 },
        "order":         "asc",
        "unit":          "km",
        "distance_type": "plane"
      }
    }
  ]
}
```

- `unit` 决定返回结果中 `sort` 值的距离单位（`km`、`m`、`mi` 等）
- `distance_type` 可选 `arc`（默认，精度高）或 `plane`（速度快，靠近两极精度下降）
- 支持对多个坐标点排序，使用 `sort_mode` 指定取最小（`min`）、最大（`max`）或平均（`avg`）距离

## 按距离打分

如果距离只是排序因素之一（还需结合全文匹配、流行度等），可在 `function_score` 查询中使用距离衰减函数：

```json
GET /attractions/_search
{
  "query": {
    "function_score": {
      "query": { "match": { "name": "pizza" } },
      "functions": [
        {
          "gauss": {
            "location": {
              "origin": { "lat": 40.715, "lon": -73.998 },
              "scale": "2km"
            }
          }
        }
      ]
    }
  }
}
```

`function_score` 还可以配合 `rescore` 限制只对前 N 条结果计算距离，提升性能。

## 相关文档

- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [地理位置实践]({{< relref "/docs/features/geo-search/geo-recipes" >}})


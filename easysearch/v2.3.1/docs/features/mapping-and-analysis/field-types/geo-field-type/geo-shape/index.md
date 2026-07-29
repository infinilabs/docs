---
title: "地理形状字段类型（Geo Shape）"
date: 0001-01-01
summary: "地理形状字段类型（Geo Shape） #  geo_shape 字段类型包含地理形状，例如多边形或地理点的集合。为了索引地理形状，Easysearch 会将形状分割成三角形网格，并将每个三角形存储在 BKD 树中。这提供了 10^-7 度的精度，代表了接近完美的空间分辨率。这个过程的性能主要受到您正在索引的多边形顶点数量多少的影响。
Geo-shapes 可以用来判断查询的形状与索引的形状的关系：
 intersects：查询的形状与索引的形状有重叠（默认） disjoint：查询的形状与索引的形状完全不重叠 within：索引的形状完全被包含在查询的形状中 contains：索引的形状完全包含查询的形状   注意：Geo-shapes 不能用于计算距离、排序、打分以及聚合。如需距离排序，请使用 geo_point 字段。
 代码样例 #  创建一个带有地理形状字段类型的映射：
PUT testindex { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;location&#34;: { &#34;type&#34;: &#34;geo_shape&#34; } } } } 格式说明 #  地理形状可以用以下格式索引：
  GeoJSON  Well-Known Text (WKT)   在 GeoJSON 和 WKT 中，坐标必须在坐标数组中按照 经度, 纬度 的顺序指定。注意在这种格式中经度是在前面的。
 地理形状类型 #  下表描述了可能的地理形状类型以及它们与 GeoJSON 和 WKT 类型的关系。"
---


# 地理形状字段类型（Geo Shape）

`geo_shape` 字段类型包含地理形状，例如多边形或地理点的集合。为了索引地理形状，Easysearch 会将形状分割成三角形网格，并将每个三角形存储在 BKD 树中。这提供了 10^-7 度的精度，代表了接近完美的空间分辨率。这个过程的性能主要受到您正在索引的多边形顶点数量多少的影响。

Geo-shapes 可以用来判断查询的形状与索引的形状的关系：

- `intersects`：查询的形状与索引的形状有重叠（默认）
- `disjoint`：查询的形状与索引的形状完全不重叠
- `within`：索引的形状完全被包含在查询的形状中
- `contains`：索引的形状完全包含查询的形状

> **注意**：Geo-shapes 不能用于计算距离、排序、打分以及聚合。如需距离排序，请使用 [geo_point]({{< relref "geo-point.md" >}}) 字段。

## 代码样例

创建一个带有地理形状字段类型的映射：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}
```

## 格式说明

地理形状可以用以下格式索引：

- [GeoJSON](https://geojson.org/)
- [Well-Known Text (WKT)](https://docs.ogc.org/is/12-063r5/12-063r5.html)

> 在 GeoJSON 和 WKT 中，坐标必须在坐标数组中按照 `经度`, `纬度` 的顺序指定。注意在这种格式中经度是在前面的。

## 地理形状类型

下表描述了可能的地理形状类型以及它们与 GeoJSON 和 WKT 类型的关系。

| Easysearch 类型    | GeoJSON 类型       | WKT 类型           | 描述                                                                                                                                                                         |
| ------------------ | ------------------ | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| point              | Point              | POINT              | 由纬度和经度指定的地理点。Easysearch 使用世界大地测量系统 (WGS84) 坐标。                                                                                                     |
| linestring         | LineString         | LINESTRING         | 由两个或更多点指定的线。可以是直线或连接的线段路径。                                                                                                                         |
| polygon            | Polygon            | POLYGON            | 由坐标形式的顶点列表指定的多边形。多边形必须是闭合的，这意味着最后一个点必须与第一个点相同。因此，要创建一个 n 边形，需要 n+1 个顶点。最少需要四个顶点，这会创建一个三角形。 |
| multipoint         | MultiPoint         | MULTIPOINT         | 不连接的离散相关点的数组。                                                                                                                                                   |
| multilinestring    | MultiLineString    | MULTILINESTRING    | 线串的数组。                                                                                                                                                                 |
| multipolygon       | MultiPolygon       | MULTIPOLYGON       | 多边形的数组。                                                                                                                                                               |
| geometrycollection | GeometryCollection | GEOMETRYCOLLECTION | 可能是不同类型的地理形状的集合。                                                                                                                                             |
| envelope           | N/A                | BBOX               | 由左上和右下顶点指定的边界矩形。                                                                                                                                             |

## Point 点位

一个点代表着由经度和纬度指定的单个坐标对。

在 GeoJSON 格式中索引一个点：

```json
PUT testindex/_doc/1
{
  "location" : {
    "type" : "point",
    "coordinates" : [74.0060, 40.7128]
  }
}
```

在 WKT 格式中索引一个点数据：

```json
PUT testindex/_doc/1
{
  "location" : "POINT (74.0060 40.7128)"
}
```

## Linestring 线串

一个线串是由两个或更多点指定的线。如果点是共线的，则线串是直线。否则，线串表示由线段组成的路径。

在 GeoJSON 格式中索引一个线串：

```json
PUT testindex/_doc/2
{
  "location" : {
    "type" : "linestring",
    "coordinates" : [[74.0060, 40.7128], [71.0589, 42.3601]]
  }
}
```

在 WKT 格式中索引一个线串：

```json
PUT testindex/_doc/2
{
  "location" : "LINESTRING (74.0060 40.7128, 71.0589 42.3601)"
}
```

## Polygon 多边形

一个多边形是由坐标形式的顶点列表指定的。多边形必须是闭合的，这意味着最后一个点必须与第一个点相同。在下面的例子中，创建了一个三角形，使用了四个点。

> GeoJSON 要求您以逆时针顺序列出多边形的顶点。WKT 不对顶点顺序施加任何限制。

在 GeoJSON 格式中索引一个多边形（三角形）：

```json
PUT testindex/_doc/3
{
  "location" : {
    "type" : "polygon",
    "coordinates" : [
      [
        [74.0060, 40.7128],
        [73.7562, 42.6526],
        [71.0589, 42.3601],
        [74.0060, 40.7128]
      ]
    ]
  }
}
```

在 WKT 格式中索引一个多边形（三角形）：

```json
PUT testindex/_doc/3
{
  "location" : "POLYGON ((74.0060 40.7128, 71.0589 42.3601, 73.7562 42.6526, 74.0060 40.7128))"
}
```

多边形可以有内部的洞。在这种情况下，坐标字段将包含多个数组。第一个数组表示外部多边形，每个后续数组表示一个洞。洞表示为多边形，并指定为坐标数组。

> GeoJSON 要求您以逆时针顺序列出多边形的顶点，并以顺时针顺序列出洞的顶点。WKT 不对顶点顺序施加任何限制。

在 GeoJSON 格式中索引一个多边形（三角形）和一个三角形洞：

```json
PUT testindex/_doc/4
{
  "location" : {
    "type" : "polygon",
    "coordinates" : [
      [
        [74.0060, 40.7128],
        [73.7562, 42.6526],
        [71.0589, 42.3601],
        [74.0060, 40.7128]
      ],
      [
        [72.6734, 41.7658],
        [73.0515, 41.5582],
        [72.6506, 41.5623],
        [72.6734, 41.7658]
      ]
    ]
  }
}
```

在 WKT 格式中索引一个多边形（三角形）和一个三角形洞：

```json
PUT testindex/_doc/4
{
  "location" : "POLYGON ((74.0060 40.7128, 71.0589 42.3601, 73.7562 42.6526, 74.0060 40.7128), (72.6734 41.7658, 72.6506 41.5623, 73.0515 41.5582, 72.6734 41.7658))"
}
```

您可以通过以顺时针或逆时针顺序列出顶点来指定 Easysearch 中的多边形。这对于不跨越日期线（小于 180°）的多边形来说是有效的。然而，跨越日期线（大于 180°）的多边形可能是模糊的，因为 WKT 不对顶点顺序施加任何限制。因此，您必须通过以逆时针顺序列出顶点来指定跨越日期线的多边形。

您可以定义一个 `orientation` 方向参数来指定顶点遍历顺序：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape",
        "orientation" : "left"
      }
    }
  }
}
```

随后索引的文档可以覆盖 `orientation` 设置：

```json
PUT testindex/_doc/3
{
  "location" : {
    "type" : "polygon",
    "orientation" : "cw",
    "coordinates" : [
      [[74.0060, 40.7128],
        [71.0589, 42.3601],
        [73.7562, 42.6526],
        [74.0060, 40.7128]]
    ]
  }
}
```

## Multipoint 多点位

一个多点是由不连接的离散相关点组成的数组。

在 GeoJSON 格式中索引一个多点：

```json
PUT testindex/_doc/6
{
  "location" : {
    "type" : "multipoint",
    "coordinates" : [
      [74.0060, 40.7128],
      [71.0589, 42.3601]
    ]
  }
}
```

在 WKT 格式中索引一个多点：

```json
PUT testindex/_doc/6
{
  "location" : "MULTIPOINT (74.0060 40.7128, 71.0589 42.3601)"
}
```

## Multilinestring 多线串

一个多线串是由线串组成的数组。

在 GeoJSON 格式中索引一个多线串：

```json
PUT testindex/_doc/2
{
  "location" : {
    "type" : "multilinestring",
    "coordinates" : [
      [[74.0060, 40.7128], [71.0589, 42.3601]],
      [[73.7562, 42.6526], [72.6734, 41.7658]]
    ]
  }
}
```

在 WKT 格式中索引一个多线串：

```json
PUT testindex/_doc/2
{
  "location" : "MULTILINESTRING ((74.0060 40.7128, 71.0589 42.3601), (73.7562 42.6526, 72.6734 41.7658))"
}
```

## Multipolygon 多个多边形

多个多边形是由多边形组成的数组。在这个例子中，第一个多边形包含一个洞，第二个不包含。

在 GeoJSON 格式中索引多个多边形：

```json
PUT testindex/_doc/4
{
  "location" : {
    "type" : "multipolygon",
    "coordinates" : [
      [
        [
          [74.0060, 40.7128],
          [73.7562, 42.6526],
          [71.0589, 42.3601],
          [74.0060, 40.7128]
        ],
        [
          [73.0515, 41.5582],
          [72.6506, 41.5623],
          [72.6734, 41.7658],
          [73.0515, 41.5582]
        ]
      ],
      [
        [
          [73.9146, 40.8252],
          [73.8871, 41.0389],
          [73.6853, 40.9747],
          [73.9146, 40.8252]
        ]
      ]
    ]
  }
}
```

在 WKT 格式中索引多个多边形：

```json
PUT testindex/_doc/4
{
  "location" : "MULTIPOLYGON (((74.0060 40.7128, 71.0589 42.3601, 73.7562 42.6526, 74.0060 40.7128), (72.6734 41.7658, 72.6506 41.5623, 73.0515 41.5582, 72.6734 41.7658)), ((73.9146 40.8252, 73.6853 40.9747, 73.8871 41.0389, 73.9146 40.8252)))"
}
```

## Geometry collection 几何集合

一个几何集合是可能是不同类型的地理形状的集合。

在 GeoJSON 格式中索引一个几何集合：

```json
PUT testindex/_doc/7
{
  "location" : {
    "type": "geometrycollection",
    "geometries": [
      {
        "type": "point",
        "coordinates": [74.0060, 40.7128]
      },
      {
        "type": "linestring",
        "coordinates": [[73.7562, 42.6526], [72.6734, 41.7658]]
      }
    ]
  }
}
```

在 WKT 格式中索引一个几何集合：

```json
PUT testindex/_doc/7
{
  "location" : "GEOMETRYCOLLECTION (POINT (74.0060 40.7128), LINESTRING(73.7562 42.6526, 72.6734 41.7658))"
}
```

## Envelope 边界矩阵

一个边界矩形是由左上和右下顶点指定的。在 GeoJSON 格式中，边界矩形的坐标是 `[[minLon, maxLat], [maxLon, minLat]]`。

在 GeoJSON 格式中索引一个边界矩形：

```json
PUT testindex/_doc/2
{
  "location" : {
    "type" : "envelope",
    "coordinates" : [[71.0589, 42.3601], [74.0060, 40.7128]]
  }
}
```

在 WKT 格式中，使用 `BBOX (minLon, maxLon, maxLat, minLat)`。

在 WKT 格式中索引一个边界矩形：

```json
PUT testindex/_doc/8
{
  "location" : "BBOX (71.0589, 74.0060, 42.3601, 40.7128)"
}
```

## 参数说明

以下表格列出了地理形状字段类型接受的参数。所有参数都是可选的。

| 参数               | 描述                                                                                                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `coerce`           | 一个布尔值，指定是否自动关闭未闭合的线环。默认为 `false`。                                                                                                                      |
| `ignore_malformed` | 一个布尔值，指定是否忽略格式错误的 GeoJSON 或 WKT 地理形状，而不是抛出异常。默认为 `false`（抛出异常）。                                                                        |
| `ignore_z_value`   | 特定于具有三个坐标的点。如果 `ignore_z_value` 为 `true`，则第三个坐标不会被索引，但仍会存储在 `_source` 字段中。如果 `ignore_z_value` 为 `false`，则会抛出异常。默认为 `true`。 |
| `orientation`      | 指定地理形状坐标列表中顶点的遍历顺序。`orientation` 可以取以下值：1. RIGHT：逆时针。使用以下字符串（大写或小写）指定 RIGHT 方向：`right`、`counterclockwise`、`ccw`。 2. LEFT：顺时针。使用以下字符串（大写或小写）指定 LEFT 方向：`left`、`clockwise`、`cw`。这个值可以被单个文档覆盖。   默认为 `RIGHT`。                                                                                                                                                                |

## 精度与距离误差

在映射中可以通过两个参数控制形状的索引精度：

### 精度（precision）

`precision` 参数控制生成的 geohash 最大长度。默认精度为 `9`，等同于约 5m × 5m 的 geohash 单元。可以使用数字（geohash 长度）或距离值（如 `50m`、`2km`）：

```json
PUT /attractions
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape",
        "precision": "100m"
      }
    }
  }
}
```

> 精度越低，需要索引的单元越少，检索越快，但形状准确性越差。减少 1-2 个等级的精度就能带来明显的性能收益。

### 距离误差（distance_error_pct）

`distance_error_pct` 指定地理形状可以接受的最大错误率，默认为 `0.025`（2.5%）。这意味着大的形状（如国家）比小的形状（如纪念碑）容许更模糊的边界：

```json
PUT /areas
{
  "mappings": {
    "properties": {
      "boundary": {
        "type": "geo_shape",
        "distance_error_pct": 0.05
      }
    }
  }
}
```

> 容许更大的错误率，对应需要索引的单元就越少，索引体积和查询开销也越低。

## 预索引形状

当查询中使用的形状很大且复杂时，在每次查询中传递完整坐标代价昂贵。可以将形状预先索引到一个独立索引中，查询时引用形状 ID：

```json
PUT /shapes/_doc/amsterdam
{
  "location": {
    "type": "polygon",
    "coordinates": [[
      [4.88330,52.38617], [4.87463,52.37254],
      [4.87875,52.36369], [4.88939,52.35850],
      [4.89840,52.35755], [4.91909,52.36217],
      [4.92656,52.36594], [4.93368,52.36615],
      [4.93342,52.37275], [4.92690,52.37632],
      [4.88330,52.38617]
    ]]
  }
}
```

查询时引用预索引形状：

```json
GET /attractions/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "indexed_shape": {
          "index": "shapes",
          "id":    "amsterdam",
          "path":  "location"
        }
      }
    }
  }
}
```

## 相关文档

- [Geo Shape 查询]({{< relref "/docs/features/geo-search/geo-shape-query.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [地理位置实践]({{< relref "/docs/features/geo-search/geo-recipes" >}})


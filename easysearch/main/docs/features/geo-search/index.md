---
title: "地理位置搜索"
date: 0001-01-01
description: "GeoPoint/GeoShape 查询与聚合，以及常见场景实践。"
summary: "我们拿着纸质地图漫步城市的日子一去不复返了。得益于智能手机，我们现在总是可以知道自己所处的准确位置，也预料到网站会使用这些信息。我想知道从当前位置步行 5 分钟内可到的那些餐馆，对伦敦更大范围内的其他餐馆并不感兴趣。
但地理位置功能仅仅是 Easysearch 的冰山一角，Easysearch 的妙处在于，它让你可以把地理位置、全文搜索、结构化搜索和分析结合到一起。
例如：告诉我提到 vitello tonnato 这种食物、步行 5 分钟内可到、且晚上 11 点还营业的餐厅，然后结合用户评价、距离、价格排序。另一个例子：给我展示一幅整个城市 8 月份可用假期出租物业的地图，并计算出每个区域的平均价格。
Easysearch 提供了两种表示地理位置的方式：用纬度－经度表示的坐标点使用 geo_point 字段类型，以 GeoJSON 格式定义的复杂地理形状，使用 geo_shape 字段类型。
Geo-points 允许你找到距离另一个坐标点一定范围内的坐标点、计算出两点之间的距离来排序或进行相关性打分、或者聚合到显示在地图上的一个网格。另一方面，Geo-shapes 纯粹是用来过滤的。它们可以用来判断两个地理形状是否有重合或者某个地理形状是否完全包含了其他地理形状。
推荐阅读顺序：
  地理位置实践：按场景整理的 Geo 实践：附近的人、围栏、路径等  字段类型参考 #     字段类型 说明      geo_point 经纬度坐标点：建模、坐标格式、距离排序与打分    geo_shape 地理形状：GeoJSON/WKT、精度控制、空间关系查询    Geohash 编码 Geohash 编码原理、精度级别与网格聚合    地理查询类型 #     查询类型 说明 字段类型      geo_bounding_box 矩形区域内的点 geo_point    geo_distance 指定距离内的点 geo_point    geo_polygon 多边形区域内的点 geo_point    geo_shape 形状空间关系（INTERSECTS、WITHIN、DISJOINT、CONTAINS） geo_point、geo_shape    地理聚合 #  地理位置数据的可视化与统计分析："
---


我们拿着纸质地图漫步城市的日子一去不复返了。得益于智能手机，我们现在总是可以知道自己所处的准确位置，也预料到网站会使用这些信息。我想知道从当前位置步行 5 分钟内可到的那些餐馆，对伦敦更大范围内的其他餐馆并不感兴趣。

但地理位置功能仅仅是 Easysearch 的冰山一角，Easysearch 的妙处在于，它让你可以把地理位置、全文搜索、结构化搜索和分析结合到一起。

例如：告诉我提到 _vitello tonnato_ 这种食物、步行 5 分钟内可到、且晚上 11 点还营业的餐厅，然后结合用户评价、距离、价格排序。另一个例子：给我展示一幅整个城市 8 月份可用假期出租物业的地图，并计算出每个区域的平均价格。

Easysearch 提供了两种表示地理位置的方式：用纬度－经度表示的坐标点使用 `geo_point` 字段类型，以 GeoJSON 格式定义的复杂地理形状，使用 `geo_shape` 字段类型。

Geo-points 允许你找到距离另一个坐标点一定范围内的坐标点、计算出两点之间的距离来排序或进行相关性打分、或者聚合到显示在地图上的一个网格。另一方面，Geo-shapes 纯粹是用来过滤的。它们可以用来判断两个地理形状是否有重合或者某个地理形状是否完全包含了其他地理形状。

推荐阅读顺序：

1. [地理位置实践]({{< relref "geo-recipes" >}})：按场景整理的 Geo 实践：附近的人、围栏、路径等

## 字段类型参考

| 字段类型 | 说明 |
|----------|------|
| [geo_point]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geo-point" >}}) | 经纬度坐标点：建模、坐标格式、距离排序与打分 |
| [geo_shape]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geo-shape" >}}) | 地理形状：GeoJSON/WKT、精度控制、空间关系查询 |
| [Geohash 编码]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geohash" >}}) | Geohash 编码原理、精度级别与网格聚合 |

## 地理查询类型

| 查询类型 | 说明 | 字段类型 |
|----------|------|----------|
| [geo_bounding_box]({{< relref "geo-bounding-box" >}}) | 矩形区域内的点 | `geo_point` |
| [geo_distance]({{< relref "geo-distance" >}}) | 指定距离内的点 | `geo_point` |
| [geo_polygon]({{< relref "geo-polygon" >}}) | 多边形区域内的点 | `geo_point` |
| [geo_shape]({{< relref "geo-shape-query" >}}) | 形状空间关系（INTERSECTS、WITHIN、DISJOINT、CONTAINS） | `geo_point`、`geo_shape` |

## 地理聚合

地理位置数据的可视化与统计分析：

- [Geo Distance 聚合]({{< relref "/docs/features/aggregations/bucket-aggregations/geo-distance" >}})：按距离范围分组
- [Geohash Grid 聚合]({{< relref "/docs/features/aggregations/bucket-aggregations/geohash-grid" >}})：按 Geohash 网格分组（热力图）
- [Geotile Grid 聚合]({{< relref "/docs/features/aggregations/bucket-aggregations/geotile-grid" >}})：按地图瓦片分组（Web 地图）
- [Geo Bounds 聚合]({{< relref "/docs/features/aggregations/metric-aggregations/geobounds" >}})：计算边界框
- [Geo Centroid 聚合]({{< relref "/docs/features/aggregations/metric-aggregations/geocentroid" >}})：计算中心点



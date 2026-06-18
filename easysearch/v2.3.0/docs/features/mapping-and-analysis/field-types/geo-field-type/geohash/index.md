---
title: "Geohash 编码"
date: 0001-01-01
description: "Geohash 编码原理、精度级别、前缀特性与在地理搜索中的应用。"
summary: "Geohash 编码 #   Geohash 是一种将经纬度坐标（lat/lon）编码成字符串的方式。这么做的初衷只是为了让地理位置在 URL 上呈现的形式更加友好，但现在 Geohash 已经变成一种在数据库中高效索引地理坐标点和地理形状的方式。
 工作原理 #  Geohash 把整个世界分为 32 个单元的格子——4 行 8 列——每一个格子都用一个字母或者数字标识。比如 g 这个单元覆盖了半个格林兰、冰岛的全部和大不列颠的大部分。
每一个单元还可以进一步被分解成新的 32 个单元，这些单元又可以继续被分解成 32 个更小的单元，不断重复下去：
 gc 覆盖了爱尔兰和英格兰 gcp 覆盖了伦敦的大部分和部分南英格兰 gcpuuz94k 是白金汉宫的入口，精确到约 5 米  核心特性：Geohash 的长度越长，精度就越高。如果两个 Geohash 有一个共同的前缀（如 gcpuuz），就表示它们挨得很近。共同的前缀越长，距离就越近。
 精度级别 #  下表展示了不同长度 Geohash 对应的近似尺寸：
   Geohash 级别 近似尺寸     g 1 ~ 5,004km × 5,004km   gc 2 ~ 1,251km × 625km   gcp 3 ~ 156km × 156km   gcpu 4 ~ 39km × 19."
---


# Geohash 编码

[Geohash](http://en.wikipedia.org/wiki/Geohash) 是一种将经纬度坐标（lat/lon）编码成字符串的方式。这么做的初衷只是为了让地理位置在 URL 上呈现的形式更加友好，但现在 Geohash 已经变成一种在数据库中高效索引地理坐标点和地理形状的方式。

---

## 工作原理

Geohash 把整个世界分为 32 个单元的格子——4 行 8 列——每一个格子都用一个字母或者数字标识。比如 `g` 这个单元覆盖了半个格林兰、冰岛的全部和大不列颠的大部分。

每一个单元还可以进一步被分解成新的 32 个单元，这些单元又可以继续被分解成 32 个更小的单元，不断重复下去：

- `gc` 覆盖了爱尔兰和英格兰
- `gcp` 覆盖了伦敦的大部分和部分南英格兰
- `gcpuuz94k` 是白金汉宫的入口，精确到约 5 米

**核心特性**：Geohash 的长度越长，精度就越高。如果两个 Geohash 有一个共同的前缀（如 `gcpuuz`），就表示它们挨得很近。共同的前缀越长，距离就越近。

---

## 精度级别

下表展示了不同长度 Geohash 对应的近似尺寸：

| Geohash | 级别 | 近似尺寸 |
|---------|------|----------|
| `g` | 1 | ~ 5,004km × 5,004km |
| `gc` | 2 | ~ 1,251km × 625km |
| `gcp` | 3 | ~ 156km × 156km |
| `gcpu` | 4 | ~ 39km × 19.5km |
| `gcpuu` | 5 | ~ 4.9km × 4.9km |
| `gcpuuz` | 6 | ~ 1.2km × 610m |
| `gcpuuz9` | 7 | ~ 152.8m × 152.8m |
| `gcpuuz94` | 8 | ~ 38.2m × 19.1m |
| `gcpuuz94k` | 9 | ~ 4.78m × 4.78m |
| `gcpuuz94kk` | 10 | ~ 1.19m × 0.60m |
| `gcpuuz94kkp` | 11 | ~ 14.9cm × 14.9cm |
| `gcpuuz94kkp5` | 12 | ~ 3.7cm × 1.8cm |

> **选择精度**：大多数应用场景中，精度 5~7（约 5km 到 150m）足以满足需求。过高的精度会增加索引体积和查询开销。

---

## 边界问题

Geohash 有一个重要的边界特性需要注意：**两个刚好相邻的位置，可能会有完全不同的 Geohash**。

例如，伦敦千禧穹顶（Millennium Dome）的 Geohash 是 `u10hbp`，因为它落在了 `u` 这个单元里，而紧挨着它东边的最大的单元是 `g`。这意味着：

- 共同前缀 ≠ 唯一的邻近判断方式
- 在网格边界处，相邻位置可能属于完全不同的 Geohash 前缀

---

## 在 Easysearch 中的应用

### 自动索引 Geohash 前缀

当你索引一个 `geo_point` 字段时，Easysearch 会自动索引该坐标点的 Geohash 以及所有的 Geohash **前缀**。

例如，索引白金汉宫入口位置（纬度 `51.501568`，经度 `-0.141257`）时，会同时索引：`g`、`gc`、`gcp`、`gcpu`、`gcpuu`、`gcpuuz`、`gcpuuz9`、`gcpuuz94`、`gcpuuz94k` 等所有前缀。

这使得基于 Geohash 前缀的聚合和过滤非常高效。

### Geohash 网格聚合

`geohash_grid` 聚合利用 Geohash 将地理坐标点聚合到网格中，非常适合地图可视化：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 5
      }
    }
  }
}
```

返回结果中每个 bucket 的 `key` 就是 Geohash 值，可以用来在地图上绘制网格热力图。

### 配合边界框优化性能

由于 Geohash 网格聚合会为所有匹配文档生成 bucket，在大数据集上可能产生大量 bucket。推荐配合 `geo_bounding_box` 过滤器限制范围：

```json
GET /locations/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": { "lat": 40.8, "lon": -74.1 },
            "bottom_right": { "lat": 40.4, "lon": -73.7 }
          }
        }
      }
    }
  },
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 6
      }
    }
  }
}
```

---

## Geohash vs Geotile

Easysearch 同时支持 `geohash_grid` 和 `geotile_grid` 两种地理网格聚合，它们使用不同的网格划分方式：

| 特性 | geohash_grid | geotile_grid |
|------|--------------|--------------|
| 网格类型 | Geohash 编码（32 进制） | 地图瓦片坐标（x/y/zoom） |
| 适用场景 | 通用地理聚合 | Web 地图瓦片渲染 |
| 精度参数 | 1-12（字符串长度） | 0-29（zoom 级别） |
| 与地图库集成 | 需要转换 | 直接对应瓦片坐标 |

如果你的应用需要与 Web 地图（如 Google Maps、Leaflet、Mapbox）集成，`geotile_grid` 可能是更好的选择。

---

## 实用技巧

### 1. 根据地图缩放级别选择精度

```
缩放级别 3-5   → precision 2-3（国家/省级）
缩放级别 6-8   → precision 4-5（城市级）
缩放级别 9-12  → precision 6-7（街区级）
缩放级别 13+   → precision 8+（街道/建筑级）
```

### 2. 获取 Geohash 对应的边界框

前端可以使用 JavaScript 库（如 `ngeohash`）将 Geohash 转换为边界框坐标，用于地图绘制：

```javascript
const geohash = require('ngeohash');
const bbox = geohash.decode_bbox('gcpuu');
// 返回 [lat_min, lon_min, lat_max, lon_max]
```

### 3. 结合 geo_bounds 聚合

使用 `geo_bounds` 子聚合获取每个 Geohash 网格内的精确边界：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 5
      },
      "aggs": {
        "cell_bounds": {
          "geo_bounds": {
            "field": "location"
          }
        }
      }
    }
  }
}
```

---

## 相关文档

- [Geo Point 字段类型]({{< relref "geo-point.md" >}})
- [Geo Shape 字段类型]({{< relref "geo-shape.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [地理位置实践]({{< relref "/docs/features/geo-search/geo-recipes" >}})
- [Geohash 网格聚合]({{< relref "/docs/features/aggregations/bucket-aggregations/geohash-grid" >}})
- [Geotile 网格聚合]({{< relref "/docs/features/aggregations/bucket-aggregations/geotile-grid" >}})


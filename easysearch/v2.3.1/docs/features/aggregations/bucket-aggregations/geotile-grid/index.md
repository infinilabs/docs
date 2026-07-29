---
title: "Geotile 网格聚合"
date: 0001-01-01
summary: "Geotile 网格聚合 #  geotile_grid 聚合将 geo_point 值按照地图瓦片坐标分组。与 geohash_grid 类似，但使用的是 Web 地图标准的瓦片坐标系统（x/y/zoom），这使得它与 Google Maps、OpenStreetMap、Mapbox 等瓦片地图服务的集成更加方便。
相关指南（先读这些） #    聚合基础教程  地理位置搜索  Geohash Grid 聚合   瓦片坐标系统 #  Web 地图使用 Slippy Map 瓦片坐标系统：
 zoom：缩放级别（0-29），级别越高网格越精细 x：水平瓦片索引（从左到右） y：垂直瓦片索引（从上到下）  瓦片标识符格式：{zoom}/{x}/{y}，如 14/8532/5765
缩放级别与覆盖范围 #     Zoom 瓦片数量 每个瓦片覆盖范围     0 1 整个世界   1 4 ~ 20,000km   5 1,024 ~ 1,250km   10 ~100万 ~ 39km   15 ~10亿 ~ 1."
---


# Geotile 网格聚合

`geotile_grid` 聚合将 `geo_point` 值按照地图瓦片坐标分组。与 `geohash_grid` 类似，但使用的是 Web 地图标准的瓦片坐标系统（x/y/zoom），这使得它与 Google Maps、OpenStreetMap、Mapbox 等瓦片地图服务的集成更加方便。

## 相关指南（先读这些）

- [聚合基础教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [Geohash Grid 聚合]({{< relref "geohash-grid" >}})

---

## 瓦片坐标系统

Web 地图使用 [Slippy Map](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames) 瓦片坐标系统：

- **zoom**：缩放级别（0-29），级别越高网格越精细
- **x**：水平瓦片索引（从左到右）
- **y**：垂直瓦片索引（从上到下）

瓦片标识符格式：`{zoom}/{x}/{y}`，如 `14/8532/5765`

### 缩放级别与覆盖范围

| Zoom | 瓦片数量 | 每个瓦片覆盖范围 |
|------|----------|------------------|
| 0 | 1 | 整个世界 |
| 1 | 4 | ~ 20,000km |
| 5 | 1,024 | ~ 1,250km |
| 10 | ~100万 | ~ 39km |
| 15 | ~10亿 | ~ 1.2km |
| 20 | ~1万亿 | ~ 38m |

---

## 参数说明

| 参数 | 必需/可选 | 类型 | 说明 |
|------|-----------|------|------|
| `field` | 必需 | String | 包含 `geo_point` 值的字段名 |
| `precision` | 可选 | Integer | 缩放级别（0-29）。默认 `7` |
| `bounds` | 可选 | Object | 限制聚合的地理边界框 |
| `size` | 可选 | Integer | 返回的最大 bucket 数量。默认 `10000` |
| `shard_size` | 可选 | Integer | 每个分片返回的 bucket 数量 |

---

## 基础用法

### 简单瓦片网格聚合

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "tiles": {
      "geotile_grid": {
        "field": "location",
        "precision": 8
      }
    }
  }
}
```

### 响应示例

```json
{
  "aggregations": {
    "tiles": {
      "buckets": [
        {
          "key": "8/131/84",
          "doc_count": 256
        },
        {
          "key": "8/131/85",
          "doc_count": 189
        },
        {
          "key": "8/132/84",
          "doc_count": 145
        }
      ]
    }
  }
}
```

每个 bucket 的 `key` 是瓦片坐标 `{zoom}/{x}/{y}`，可以直接用于请求对应的地图瓦片。

---

## 配合边界框过滤

限制在特定区域内聚合：

```json
GET /locations/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": {
              "lat": 40.8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.4,
              "lon": -73.7
            }
          }
        }
      }
    }
  },
  "aggs": {
    "nyc_tiles": {
      "geotile_grid": {
        "field": "location",
        "precision": 12
      }
    }
  }
}
```

---

## 使用 bounds 参数

直接在聚合中指定边界范围：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "tiles": {
      "geotile_grid": {
        "field": "location",
        "precision": 10,
        "bounds": {
          "top_left": { "lat": 40.8, "lon": -74.1 },
          "bottom_right": { "lat": 40.4, "lon": -73.7 }
        }
      }
    }
  }
}
```

---

## 嵌套子聚合

### 每个瓦片的中心点

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "tiles": {
      "geotile_grid": {
        "field": "location",
        "precision": 10
      },
      "aggs": {
        "centroid": {
          "geo_centroid": {
            "field": "location"
          }
        }
      }
    }
  }
}
```

### 每个瓦片的统计信息

```json
GET /real_estate/_search
{
  "size": 0,
  "aggs": {
    "tiles": {
      "geotile_grid": {
        "field": "location",
        "precision": 12
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "price_stats": {
          "stats": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

---

## 在 Composite 聚合中使用

`geotile_grid` 可以作为 `composite` 聚合的源，实现分页遍历所有瓦片：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "tiles": {
      "composite": {
        "size": 100,
        "sources": [
          {
            "tile": {
              "geotile_grid": {
                "field": "location",
                "precision": 8
              }
            }
          }
        ]
      }
    }
  }
}
```

翻页时使用 `after` 参数：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "tiles": {
      "composite": {
        "size": 100,
        "sources": [
          {
            "tile": {
              "geotile_grid": {
                "field": "location",
                "precision": 8
              }
            }
          }
        ],
        "after": {
          "tile": "8/131/85"
        }
      }
    }
  }
}
```

---

## 与地图库集成

### 转换瓦片坐标为边界框

前端可以使用标准公式将瓦片坐标转换为经纬度边界：

```javascript
function tile2boundingBox(z, x, y) {
  const n = Math.PI - 2 * Math.PI * y / Math.pow(2, z);
  return {
    north: 180 / Math.PI * Math.atan(0.5 * (Math.exp(n) - Math.exp(-n))),
    south: 180 / Math.PI * Math.atan(0.5 * (
      Math.exp(Math.PI - 2 * Math.PI * (y + 1) / Math.pow(2, z)) - 
      Math.exp(-(Math.PI - 2 * Math.PI * (y + 1) / Math.pow(2, z)))
    )),
    west: x / Math.pow(2, z) * 360 - 180,
    east: (x + 1) / Math.pow(2, z) * 360 - 180
  };
}

// 解析瓦片 key
function parseTileKey(key) {
  const [z, x, y] = key.split('/').map(Number);
  return { z, x, y };
}
```

### 在 Leaflet 中显示聚合结果

```javascript
// 假设 response 是 Easysearch 返回的聚合结果
const buckets = response.aggregations.tiles.buckets;

buckets.forEach(bucket => {
  const { z, x, y } = parseTileKey(bucket.key);
  const bounds = tile2boundingBox(z, x, y);
  
  L.rectangle([
    [bounds.south, bounds.west],
    [bounds.north, bounds.east]
  ], {
    color: getColorByCount(bucket.doc_count),
    weight: 1,
    fillOpacity: 0.5
  }).bindPopup(`数量: ${bucket.doc_count}`).addTo(map);
});
```

---

## 性能优化

### 1. 根据地图缩放级别动态调整精度

```javascript
function getPrecisionForZoom(mapZoom) {
  // 瓦片精度通常与地图缩放级别相同或略低
  return Math.min(mapZoom, 15);
}
```

### 2. 限制返回数量

对于高精度查询，限制返回的瓦片数量：

```json
{
  "aggs": {
    "tiles": {
      "geotile_grid": {
        "field": "location",
        "precision": 15,
        "size": 1000
      }
    }
  }
}
```

### 3. 配合边界框使用

始终使用 `bounds` 或 `geo_bounding_box` 过滤器，避免扫描全量数据。

---

## geotile_grid vs geohash_grid

| 特性 | geotile_grid | geohash_grid |
|------|--------------|--------------|
| 坐标系统 | Web 瓦片 (x/y/zoom) | Geohash 编码 |
| 精度范围 | 0-29 | 1-12 |
| Key 格式 | `8/131/84` | `dr5rs` |
| 地图集成 | ✅ 直接对应瓦片 | ⚠️ 需要转换 |
| 网格形状 | 正方形（墨卡托投影） | 矩形（可变） |

**选择建议**：
- 与 Web 地图服务集成 → `geotile_grid`
- 通用地理聚合 → `geohash_grid`
- 需要与第三方 Geohash 服务交互 → `geohash_grid`

---

## 相关文档

- [Geohash Grid 聚合]({{< relref "geohash-grid" >}})
- [Geo Bounds 聚合]({{< relref "/docs/features/aggregations/metric-aggregations/geobounds" >}})
- [Geo Centroid 聚合]({{< relref "/docs/features/aggregations/metric-aggregations/geocentroid" >}})
- [Composite 聚合]({{< relref "composite" >}})
- [Geohash 编码原理]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geohash.md" >}})


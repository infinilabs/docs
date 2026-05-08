---
title: "Geohash 网格聚合"
date: 0001-01-01
summary: "Geohash 网格聚合 #  geohash_grid 聚合将 geo_point 值按照 Geohash 编码分组到网格单元中。这对于在地图上可视化大量地理坐标点非常有用——将密集的点聚合成可管理的网格，每个网格显示该区域的统计信息。
相关指南（先读这些） #    聚合基础教程  地理位置搜索  Geohash 编码原理   参数说明 #     参数 必需/可选 类型 说明     field 必需 String 包含 geo_point 值的字段名   precision 可选 Integer/String Geohash 精度级别（1-12）或距离字符串（如 1km）。默认 5   bounds 可选 Object 限制聚合的地理边界框，只处理该范围内的点   size 可选 Integer 返回的最大 bucket 数量。默认 10000   shard_size 可选 Integer 每个分片返回的 bucket 数量，用于提高精度。默认 max(10, size)    精度级别参考 #     精度 网格尺寸 适用场景     1 ~ 5,000km 大洲级别   2 ~ 1,250km 国家级别   3 ~ 156km 大区域   4 ~ 39km 城市群   5 ~ 4."
---


# Geohash 网格聚合

`geohash_grid` 聚合将 `geo_point` 值按照 Geohash 编码分组到网格单元中。这对于在地图上可视化大量地理坐标点非常有用——将密集的点聚合成可管理的网格，每个网格显示该区域的统计信息。

## 相关指南（先读这些）

- [聚合基础教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [Geohash 编码原理]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geohash.md" >}})

---

## 参数说明

| 参数 | 必需/可选 | 类型 | 说明 |
|------|-----------|------|------|
| `field` | 必需 | String | 包含 `geo_point` 值的字段名 |
| `precision` | 可选 | Integer/String | Geohash 精度级别（1-12）或距离字符串（如 `1km`）。默认 `5` |
| `bounds` | 可选 | Object | 限制聚合的地理边界框，只处理该范围内的点 |
| `size` | 可选 | Integer | 返回的最大 bucket 数量。默认 `10000` |
| `shard_size` | 可选 | Integer | 每个分片返回的 bucket 数量，用于提高精度。默认 `max(10, size)` |

### 精度级别参考

| 精度 | 网格尺寸 | 适用场景 |
|------|----------|----------|
| 1 | ~ 5,000km | 大洲级别 |
| 2 | ~ 1,250km | 国家级别 |
| 3 | ~ 156km | 大区域 |
| 4 | ~ 39km | 城市群 |
| 5 | ~ 4.9km | 城市/区县 |
| 6 | ~ 1.2km | 街区 |
| 7 | ~ 153m | 街道 |
| 8 | ~ 38m | 建筑群 |

---

## 基础用法

### 简单网格聚合

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

### 响应示例

```json
{
  "aggregations": {
    "grid": {
      "buckets": [
        {
          "key": "dr5rs",
          "doc_count": 125
        },
        {
          "key": "dr5re",
          "doc_count": 89
        },
        {
          "key": "dr5ru",
          "doc_count": 67
        }
      ]
    }
  }
}
```

每个 bucket 的 `key` 是 Geohash 值，可以用前端库解码为边界框坐标，在地图上绘制网格。

---

## 配合边界框过滤

为避免在大数据集上生成过多 bucket，推荐配合 `geo_bounding_box` 查询限制范围：

```json
GET /attractions/_search
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
    "new_york_grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 6
      }
    }
  }
}
```

---

## 使用 bounds 参数

可以直接在聚合中使用 `bounds` 参数限制聚合的地理范围，效果更好：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 6,
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

可以在每个网格单元内计算更多统计信息：

### 每个网格的平均评分

```json
GET /restaurants/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 5
      },
      "aggs": {
        "avg_rating": {
          "avg": {
            "field": "rating"
          }
        },
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

### 每个网格的中心点

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

### 每个网格的边界框

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

## 按距离指定精度

除了数字精度级别，还可以使用距离字符串：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": "1km"
      }
    }
  }
}
```

Easysearch 会自动选择最接近指定距离的精度级别。

---

## 高基数场景处理

### 控制返回数量

默认最多返回 10000 个 bucket。如果网格数量超过限制，只返回文档数最多的 bucket：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 7,
        "size": 500
      }
    }
  }
}
```

### 提高精度

使用 `shard_size` 提高聚合精度（会增加内存消耗）：

```json
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 6,
        "size": 100,
        "shard_size": 500
      }
    }
  }
}
```

---

## 地图可视化示例

### 热力图数据准备

```json
GET /check_ins/_search
{
  "size": 0,
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-7d/d",
        "lte": "now"
      }
    }
  },
  "aggs": {
    "heatmap": {
      "geohash_grid": {
        "field": "location",
        "precision": 6
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

前端可以将结果转换为热力图数据：

```javascript
const heatmapData = response.aggregations.heatmap.buckets.map(bucket => ({
  lat: bucket.centroid.location.lat,
  lng: bucket.centroid.location.lon,
  weight: bucket.doc_count
}));
```

---

## 与 geotile_grid 的对比

| 特性 | geohash_grid | geotile_grid |
|------|--------------|--------------|
| 网格类型 | Geohash 编码 | 地图瓦片坐标 |
| 精度参数 | 1-12 | 0-29 (zoom) |
| 与 Web 地图集成 | 需要转换 | 直接对应瓦片 |
| 适用场景 | 通用聚合 | 瓦片地图渲染 |

如果你的地图使用标准瓦片服务（如 Google Maps、OpenStreetMap），`geotile_grid` 可能更方便。

---

## 相关文档

- [Geohash 编码原理]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geohash.md" >}})
- [Geotile Grid 聚合]({{< relref "geotile-grid" >}})
- [Geo Bounds 聚合]({{< relref "/docs/features/aggregations/metric-aggregations/geobounds" >}})
- [Geo Centroid 聚合]({{< relref "/docs/features/aggregations/metric-aggregations/geocentroid" >}})
- [Geo Distance 聚合]({{< relref "geo-distance" >}})


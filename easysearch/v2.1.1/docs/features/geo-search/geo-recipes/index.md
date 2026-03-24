---
title: "地理位置实践"
date: 0001-01-01
description: "按场景整理的 Geo 实践：附近的人、围栏、路径等。"
summary: "地理位置实践 #  本页以“场景实践”的形式给出几个常见地理位置需求下的解决方案，帮助你在实际应用中更好地使用地理位置功能。
在具体 API 上，Easysearch 主要通过：
 geo_distance / _geo_distance：按距离过滤与排序 geo_shape：做复杂形状的围栏与空间关系判断 geohash_grid / geo_bounds：在地图上做网格聚合与视野控制  下面的配方会围绕这些能力展开。
地理位置聚合 #  虽然按照地理位置对结果进行过滤或者打分很有用，但是在地图上呈现信息给用户通常更加有用。一个查询可能会返回太多结果以至于不能单独地展现每一个地理坐标点，但是地理位置聚合可以用来将地理坐标聚集到更加容易管理的 buckets 中。
处理 geo_point 类型字段的三种聚合：
地理距离聚合 #  geo_distance 聚合对一些搜索非常有用，例如找到所有距离我 1km 以内的披萨店。搜索结果应该也的确被限制在用户指定 1km 范围内，但是我们可以添加在 2km 范围内找到的其他结果：
GET /attractions/_search { &#34;query&#34;: { &#34;bool&#34;: { &#34;must&#34;: { &#34;match&#34;: { &#34;name&#34;: &#34;pizza&#34; } }, &#34;filter&#34;: { &#34;geo_bounding_box&#34;: { &#34;location&#34;: { &#34;top_left&#34;: { &#34;lat&#34;: 40.8, &#34;lon&#34;: -74.1 }, &#34;bottom_right&#34;: { &#34;lat&#34;: 40.4, &#34;lon&#34;: -73."
---


# 地理位置实践

本页以“场景实践”的形式给出几个常见地理位置需求下的解决方案，帮助你在实际应用中更好地使用地理位置功能。

在具体 API 上，Easysearch 主要通过：

- `geo_distance` / `_geo_distance`：按距离过滤与排序
- `geo_shape`：做复杂形状的围栏与空间关系判断
- `geohash_grid` / `geo_bounds`：在地图上做网格聚合与视野控制

下面的配方会围绕这些能力展开。

## 地理位置聚合

虽然按照地理位置对结果进行过滤或者打分很有用，但是在地图上呈现信息给用户通常更加有用。一个查询可能会返回太多结果以至于不能单独地展现每一个地理坐标点，但是地理位置聚合可以用来将地理坐标聚集到更加容易管理的 buckets 中。

处理 `geo_point` 类型字段的三种聚合：

### 地理距离聚合

`geo_distance` 聚合对一些搜索非常有用，例如找到所有距离我 1km 以内的披萨店。搜索结果应该也的确被限制在用户指定 1km 范围内，但是我们可以添加在 2km 范围内找到的其他结果：

```json
GET /attractions/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "name": "pizza"
        }
      },
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": {
              "lat":  40.8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.7
            }
          }
        }
      }
    }
  },
  "aggs": {
    "per_ring": {
      "geo_distance": {
        "field":    "location",
        "unit":     "km",
        "origin": {
          "lat":    40.712,
          "lon":   -73.988
        },
        "ranges": [
          { "from": 0, "to": 1 },
          { "from": 1, "to": 2 }
        ]
      }
    }
  },
  "post_filter": {
    "geo_distance": {
      "distance":   "1km",
      "location": {
        "lat":      40.712,
        "lon":     -73.988
      }
    }
  }
}
```

主查询查找名称中含有 `pizza` 的饭店。`geo_bounding_box` 筛选那些只在纽约区域的结果。`geo_distance` 聚合统计距离用户 1km 以内，1km 到 2km 的结果的数量。最后，`post_filter` 将结果缩小至那些在用户 1km 范围内的饭店。

在这个例子中，我们计算了落在每个同心环内的饭店数量。当然，我们可以在 `per_rings` 聚合下面嵌套子聚合来计算每个环的平均价格、最受欢迎程度，等等。

### Geohash 网格聚合

`geohash_grid` 聚合将文档按照 geohash 范围来分组，用来显示在地图上。这对于在地图上可视化大量地理坐标点非常有用。

```json
GET /attractions/_search
{
  "size": 0,
  "aggs": {
    "world": {
      "geohash_grid": {
        "field":     "location",
        "precision": 2
      }
    }
  }
}
```

`precision` 参数控制 geohash 的精度级别。精度越高，网格越细，但聚合的 buckets 也越多。

### 地理边界聚合

`geo_bounds` 聚合返回一个包含所有地理位置坐标点的边界的经纬度坐标，这对显示地图时缩放比例的选择非常有用。

```json
GET /attractions/_search
{
  "size": 0,
  "aggs": {
    "viewport": {
      "geo_bounds": {
        "field": "location"
      }
    }
  }
}
```

返回结果会包含一个边界框，可以用来设置地图的初始视图。

## 常见场景实践

### 场景一：附近的人/商家

需求：找到距离用户当前位置一定范围内的商家或用户。

方案：

1. 使用 `geo_distance` 过滤器限制距离范围
2. 使用 `_geo_distance` 排序按距离排序
3. 或者使用 `function_score` 结合距离衰减函数来影响评分

示例：

```json
GET /businesses/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_distance": {
          "distance": "5km",
          "location": {
            "lat": 40.715,
            "lon": -73.988
          }
        }
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location": {
          "lat": 40.715,
          "lon": -73.988
        },
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
```

### 场景二：地理围栏

需求：判断某个位置是否在指定的地理区域内。

方案：

1. 使用 `geo_shape` 查询判断点是否在形状内
2. 对于复杂形状，使用预索引形状提高性能

示例：

```json
GET /locations/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "relation": "within",
        "shape": {
          "type": "polygon",
          "coordinates": [[
            [4.88330,52.38617],
            [4.87463,52.37254],
            [4.87875,52.36369],
            [4.88939,52.35850],
            [4.89840,52.35755],
            [4.91909,52.36217],
            [4.92656,52.36594],
            [4.93368,52.36615],
            [4.93342,52.37275],
            [4.92690,52.37632],
            [4.88330,52.38617]
          ]]
        }
      }
    }
  }
}
```

### 场景三：地图可视化

需求：在地图上展示大量地理坐标点的分布情况。

方案：

1. 使用 `geohash_grid` 聚合将点聚合到网格中
2. 使用 `geo_bounds` 聚合获取边界框设置地图视图
3. 使用 `geo_distance` 聚合按距离范围分组

示例：

```json
GET /attractions/_search
{
  "size": 0,
  "aggs": {
    "map_view": {
      "geo_bounds": {
        "field": "location"
      }
    },
    "grid": {
      "geohash_grid": {
        "field":     "location",
        "precision": 6
      },
      "aggs": {
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

### 场景四：结合全文搜索和地理位置

需求：搜索附近的餐厅，同时考虑名称匹配度和距离。

方案：

1. 使用 `bool` 查询结合全文搜索和地理位置过滤
2. 使用 `function_score` 结合文本相关度和距离衰减

示例：

```json
GET /restaurants/_search
{
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": {
            "match": {
              "name": "pizza"
            }
          },
          "filter": {
            "geo_distance": {
              "distance": "10km",
              "location": {
                "lat": 40.715,
                "lon": -73.988
              }
            }
          }
        }
      },
      "functions": [
        {
          "gauss": {
            "location": {
              "origin": {
                "lat": 40.715,
                "lon": -73.988
              },
              "offset": "2km",
              "scale":  "3km"
            }
          }
        }
      ],
      "score_mode": "multiply"
    }
  }
}
```

## 性能优化建议

1. **优先使用 geo_bounding_box**：如果精度要求不高，使用 `geo_bounding_box` 比 `geo_distance` 更高效
2. **先用 bounding_box 缩小范围**：对大数据集，先用矩形框过滤大幅减少候选集，再用 `geo_distance` 精确筛选
3. **选择合适的距离计算方式**：根据精度要求选择合适的 `distance_type`（`arc` 默认精度高，`plane` 更快但靠近两极精度下降）
4. **预索引复杂形状**：对于复杂的地理形状，使用预索引形状可以避免每次查询都传递大量坐标
5. **合理设置精度**：对于 geo_shape，根据实际需求设置合适的精度和距离误差，避免过度索引

## 小结

- Geohashes 是一种将经纬度编码成字符串的方式，可以用于高效索引
- 地理位置聚合可以用于地图可视化和数据分析
- 常见场景包括附近的人/商家、地理围栏、地图可视化等
- 可以结合全文搜索和地理位置来提供更好的搜索体验
- 性能优化包括选择合适的过滤器、使用索引优化、预索引形状等

下一步可以继续阅读：

- [Geo Point 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geo-point" >}})
- [Geo Shape 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geo-shape" >}})

## 参考手册（API 与参数）

- [geo_bounding_box 查询]({{< relref "/docs/features/geo-search/geo-bounding-box.md" >}})
- [geo_distance 查询]({{< relref "/docs/features/geo-search/geo-distance.md" >}})
- [geo_polygon 查询]({{< relref "/docs/features/geo-search/geo-polygon.md" >}})
- [geo_shape 查询]({{< relref "/docs/features/geo-search/geo-shape-query.md" >}})
- [GeoPoint 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geo-point.md" >}})
- [GeoShape 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/geo-field-type/geo-shape.md" >}})



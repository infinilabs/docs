---
title: "地理距离聚合（Geo Distance）"
date: 0001-01-01
summary: "地理距离聚合 #  geo_distance 聚合根据与一个起始 geo_point 字段距离将文档分组到同心圆中。它与 range 聚合相同，只是它作用于地理位置。
相关指南（先读这些） #    聚合基础  地理位置搜索  Geo 场景实践  例如，你可以使用 geo_distance 聚合来查找你 1 公里范围内的所有披萨店。搜索结果仅限于你指定的 1 公里半径范围内，但你还可以添加另一个在 2 公里范围内找到的结果。
您只能对映射为 geo_point 的字段使用 geo_distance 聚合。
点是一个单一的地理坐标，例如您智能手机显示的当前位置。在 Easysearch 中，点表示如下：
{ &#34;location&#34;: { &#34;type&#34;: &#34;point&#34;, &#34;coordinates&#34;: { &#34;lat&#34;: 83.76, &#34;lon&#34;: -81.2 } } } 您还可以将纬度和经度指定为数组 [-81.20, 83.76] 或字符串 &quot;83.76, -81.20&quot;
此表列出了 geo_distance 聚合的相关字段：
   字段 必需/可选 描述     field 必需 指定您要处理的地理点字段。   origin 必需 指定用于计算距离的地理点。   ranges 必需 指定一组范围，根据文档与目标点的距离收集文档。   unit 可选 定义 ranges 数组中使用的单位。 unit 默认为 m （米），但你可以切换到其他单位，如 km （千米）、 mi （英里）、 in （英寸）、 yd （码）、 cm （厘米）和 mm （毫米）。   distance_type 可选 指定 Easysearch 如何计算距离。默认值为 sloppy_arc （更快但精度较低），也可以设置为 arc （较慢但最精确）或 plane （最快但精度最低）。由于误差范围较大，仅适用于小地理区域使用 plane 。    语法如下："
---


# 地理距离聚合

`geo_distance` 聚合根据与一个起始 `geo_point` 字段距离将文档分组到同心圆中。它与 `range` 聚合相同，只是它作用于地理位置。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [Geo 场景实践]({{< relref "/docs/features/geo-search/geo-recipes" >}})

例如，你可以使用 `geo_distance` 聚合来查找你 1 公里范围内的所有披萨店。搜索结果仅限于你指定的 1 公里半径范围内，但你还可以添加另一个在 2 公里范围内找到的结果。

您只能对映射为 `geo_point` 的字段使用 `geo_distance` 聚合。

点是一个单一的地理坐标，例如您智能手机显示的当前位置。在 Easysearch 中，点表示如下：

```
{
  "location": {
    "type": "point",
    "coordinates": {
      "lat": 83.76,
      "lon": -81.2
    }
  }
}
```

您还可以将纬度和经度指定为数组 `[-81.20, 83.76]` 或字符串 `"83.76, -81.20"`

此表列出了 `geo_distance` 聚合的相关字段：

| 字段            | 必需/可选 | 描述                                                                                                                                                                                          |
| --------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`         | 必需      | 指定您要处理的地理点字段。                                                                                                                                                                    |
| `origin`        | 必需      | 指定用于计算距离的地理点。                                                                                                                                                                    |
| `ranges`        | 必需      | 指定一组范围，根据文档与目标点的距离收集文档。                                                                                                                                                |
| `unit`          | 可选      | 定义 `ranges` 数组中使用的单位。 unit 默认为 m （米），但你可以切换到其他单位，如 km （千米）、 mi （英里）、 in （英寸）、 yd （码）、 cm （厘米）和 mm （毫米）。                           |
| `distance_type` | 可选      | 指定 Easysearch 如何计算距离。默认值为 `sloppy_arc` （更快但精度较低），也可以设置为 `arc` （较慢但最精确）或 `plane` （最快但精度最低）。由于误差范围较大，仅适用于小地理区域使用 `plane` 。 |

语法如下：

```
{
  "aggs": {
    "aggregation_name": {
      "geo_distance": {
        "field": "field_1",
        "origin": "x, y",
        "ranges": [
          {
            "to": "value_1"
          },
          {
            "from": "value_2",
            "to": "value_3"
          },
          {
            "from": "value_4"
          }
        ]
      }
    }
  }
}

```

这个示例根据与 `geo-point` 字段的以下距离形成分组：

- 少于 10 公里
- 从 10 到 20 公里
- 从 20 到 50 公里
- 从 50 到 100 公里
- 超过 100 公里

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "position": {
      "geo_distance": {
        "field": "geo.coordinates",
        "origin": {
          "lat": 83.76,
          "lon": -81.2
        },
        "ranges": [
          {
            "to": 10
          },
          {
            "from": 10,
            "to": 20
          },
          {
            "from": 20,
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}

```

返回内容

```
...
"aggregations" : {
  "position" : {
    "buckets" : [
      {
        "key" : "*-10.0",
        "from" : 0.0,
        "to" : 10.0,
        "doc_count" : 0
      },
      {
        "key" : "10.0-20.0",
        "from" : 10.0,
        "to" : 20.0,
        "doc_count" : 0
      },
      {
        "key" : "20.0-50.0",
        "from" : 20.0,
        "to" : 50.0,
        "doc_count" : 0
      },
      {
        "key" : "50.0-100.0",
        "from" : 50.0,
        "to" : 100.0,
        "doc_count" : 0
      },
      {
        "key" : "100.0-*",
        "from" : 100.0,
        "doc_count" : 14074
      }
    ]
  }
 }
}
```


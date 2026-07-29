---
title: "地理边界聚合（Geo Bounds）"
date: 0001-01-01
summary: "地理边界聚合 #  geo_bounds 聚合是一个多值指标聚合，用于计算包含一组 geo_point 或 geo_shape 对象的地理边界框。边界框以十进制编码的经纬度（lat-lon）对形式返回，作为矩形的左上角和右下角顶点。
相关指南（先读这些） #    聚合基础  地理位置搜索  Geo 场景实践  参数说明 #  geo_bounds 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 计算地理边界所使用的包含地理点或地理形状的字段的名称。   wrap_longitude 可选 Boolean 是否允许边界框与国际日期变更线重叠。默认值为 true 。    参考样例 #  以下示例查询数据中每个订单的 geo_bounds 的 geoip.location （每个 geoip.location 是一个地理点）：
GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;geo&#34;: { &#34;geo_bounds&#34;: { &#34;field&#34;: &#34;geoip."
---


# 地理边界聚合

`geo_bounds` 聚合是一个多值指标聚合，用于计算包含一组 `geo_point` 或 `geo_shape` 对象的地理边界框。边界框以十进制编码的经纬度（lat-lon）对形式返回，作为矩形的左上角和右下角顶点。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [地理位置搜索]({{< relref "/docs/features/geo-search/" >}})
- [Geo 场景实践]({{< relref "/docs/features/geo-search/geo-recipes" >}})

## 参数说明

`geo_bounds` 聚合采用以下参数。

| 参数             | 必需/可选 | 数据类型 | 描述                                                   |
| ---------------- | --------- | -------- | ------------------------------------------------------ |
| `field`          | 必需      | String   | 计算地理边界所使用的包含地理点或地理形状的字段的名称。 |
| `wrap_longitude` | 可选      | Boolean  | 是否允许边界框与国际日期变更线重叠。默认值为 true 。   |

## 参考样例

以下示例查询数据中每个订单的 `geo_bounds` 的 `geoip.location` （每个 `geoip.location` 是一个地理点）：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "geo": {
      "geo_bounds": {
        "field": "geoip.location"
      }
    }
  }
}

```

## 返回内容

如以下示例响应所示，聚合返回包含 `geoip.location` 字段中所有地理点的 `geobounds` ：

```
{
  "took": 16,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "geo": {
      "bounds": {
        "top_left": {
          "lat": 52.49999997206032,
          "lon": -118.20000001229346
        },
        "bottom_right": {
          "lat": 4.599999985657632,
          "lon": 55.299999956041574
        }
      }
    }
  }
}
```



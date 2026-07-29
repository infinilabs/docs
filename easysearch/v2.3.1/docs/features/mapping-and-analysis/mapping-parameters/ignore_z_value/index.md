---
title: "忽略Z坐标（Ignore Z Value）"
date: 0001-01-01
summary: "忽略Z坐标（Ignore Z Value） #  确定是否忽略地理空间数据中的 Z 坐标（高度）。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;location&#34;: { &#34;type&#34;: &#34;geo_point&#34;, &#34;ignore_z_value&#34;: true } } } } 适用的字段类型 #   geo_point geo_shape  默认值 #   true  说明 #  地理坐标通常是二维的（经度和纬度）。某些输入格式可能包括 Z 坐标（如 GeoJSON 中的高度）。启用此参数时，Z 坐标会被忽略。禁用时，包含 Z 坐标会导致错误。
示例 #  POST my-index/_doc/1 { &#34;location&#34;: { &#34;lat&#34;: 40.7128, &#34;lon&#34;: -74.0060, &#34;z&#34;: 100 } } 相关参数 #    索引参数（Index）  "
---


# 忽略Z坐标（Ignore Z Value）

确定是否忽略地理空间数据中的 Z 坐标（高度）。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point",
        "ignore_z_value": true
      }
    }
  }
}
```

## 适用的字段类型

- `geo_point`
- `geo_shape`

## 默认值

- `true`

## 说明

地理坐标通常是二维的（经度和纬度）。某些输入格式可能包括 Z 坐标（如 GeoJSON 中的高度）。启用此参数时，Z 坐标会被忽略。禁用时，包含 Z 坐标会导致错误。

## 示例

```json
POST my-index/_doc/1
{
  "location": {
    "lat": 40.7128,
    "lon": -74.0060,
    "z": 100
  }
}
```

## 相关参数

- [索引参数（Index）]({{< relref "index.md" >}})


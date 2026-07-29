---
title: "字段数据频率过滤（Fielddata Frequency Filter）"
date: 0001-01-01
summary: "字段数据频率过滤（Fielddata Frequency Filter） #  为了减少内存占用，可以配置字段数据过滤器来移除低频率的项。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;tags&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;fielddata&#34;: true, &#34;fielddata_frequency_filter&#34;: { &#34;min&#34;: 2, &#34;min_segment_size&#34;: 50 } } } } } 适用的字段类型 #   text（启用 fielddata 时）  参数选项 #   min: 项必须出现的最少次数（默认值：0） max: 项可能出现的最多次数（无上限） min_segment_size: 仅在至少有这么多文档的段中应用过滤器（默认值：0）  说明 #  当启用字段数据用于聚合或排序时，此参数可以降低内存占用。低频项会被从内存中移除。
相关参数 #    字段数据参数（Fielddata）  "
---


# 字段数据频率过滤（Fielddata Frequency Filter）

为了减少内存占用，可以配置字段数据过滤器来移除低频率的项。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "tags": {
        "type": "text",
        "fielddata": true,
        "fielddata_frequency_filter": {
          "min": 2,
          "min_segment_size": 50
        }
      }
    }
  }
}
```

## 适用的字段类型

- `text`（启用 `fielddata` 时）

## 参数选项

- **min**: 项必须出现的最少次数（默认值：0）
- **max**: 项可能出现的最多次数（无上限）
- **min_segment_size**: 仅在至少有这么多文档的段中应用过滤器（默认值：0）

## 说明

当启用字段数据用于聚合或排序时，此参数可以降低内存占用。低频项会被从内存中移除。

## 相关参数

- [字段数据参数（Fielddata）]({{< relref "fielddata.md" >}})


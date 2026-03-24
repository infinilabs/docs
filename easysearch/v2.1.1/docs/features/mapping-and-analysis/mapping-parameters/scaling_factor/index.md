---
title: "缩放因子（Scaling Factor）"
date: 0001-01-01
summary: "缩放因子（Scaling Factor） #  缩放比因子用于将浮点值转换为长整数以获得更高的精度。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;price&#34;: { &#34;type&#34;: &#34;scaled_float&#34;, &#34;scaling_factor&#34;: 100 } } } } 适用的字段类型 #   scaled_float（mapper-extras 模块）  说明 #  scaled_float 字段通过将浮点值乘以缩放比因子来将其存储为长整数。这可以节省磁盘空间并提高聚合性能。例如，使用 scaling_factor: 100，值 10.5 被存储为 1050。
缩放比因子必须是正数。常见值包括：
 10 - 一位小数 100 - 两位小数 1000 - 三位小数  示例 #  POST my-index/_doc/1 { &#34;price&#34;: 10.50 } 相关参数 #    索引参数（Index）  文档值参数（Doc Values）  "
---


# 缩放因子（Scaling Factor）

缩放比因子用于将浮点值转换为长整数以获得更高的精度。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      }
    }
  }
}
```

## 适用的字段类型

- `scaled_float`（mapper-extras 模块）

## 说明

`scaled_float` 字段通过将浮点值乘以缩放比因子来将其存储为长整数。这可以节省磁盘空间并提高聚合性能。例如，使用 `scaling_factor: 100`，值 `10.5` 被存储为 `1050`。

缩放比因子必须是正数。常见值包括：
- `10` - 一位小数
- `100` - 两位小数
- `1000` - 三位小数

## 示例

```json
POST my-index/_doc/1
{
  "price": 10.50
}


```

## 相关参数

- [索引参数（Index）]({{< relref "index.md" >}})
- [文档值参数（Doc Values）]({{< relref "doc_values.md" >}})


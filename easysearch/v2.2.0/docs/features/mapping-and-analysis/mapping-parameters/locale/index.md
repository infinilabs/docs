---
title: "区域设置（Locale）"
date: 0001-01-01
summary: "区域设置（Locale） #  用于解析日期或范围值的区域设置。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;publish_date&#34;: { &#34;type&#34;: &#34;date&#34;, &#34;format&#34;: &#34;dd/MM/yyyy&#34;, &#34;locale&#34;: &#34;fr_FR&#34; } } } } 适用的字段类型 #   date date_nanos date_range integer_range float_range long_range double_range  说明 #  某些日期格式取决于特定的区域设置（例如月份名称）。使用此参数指定应用于解析的区域设置。
常见区域设置值 #   en_US - 美国英语 fr_FR - 法语（法国） de_DE - 德语（德国） es_ES - 西班牙语（西班牙） zh_CN - 中文（中国）  相关参数 #    格式参数（Format）  "
---


# 区域设置（Locale）

用于解析日期或范围值的区域设置。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "publish_date": {
        "type": "date",
        "format": "dd/MM/yyyy",
        "locale": "fr_FR"
      }
    }
  }
}
```

## 适用的字段类型

- `date`
- `date_nanos`
- `date_range`
- `integer_range`
- `float_range`
- `long_range`
- `double_range`

## 说明

某些日期格式取决于特定的区域设置（例如月份名称）。使用此参数指定应用于解析的区域设置。

## 常见区域设置值

- `en_US` - 美国英语
- `fr_FR` - 法语（法国）
- `de_DE` - 德语（德国）
- `es_ES` - 西班牙语（西班牙）
- `zh_CN` - 中文（中国）

## 相关参数

- [格式参数（Format）]({{< relref "format.md" >}})


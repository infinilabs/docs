---
title: "转换处理器"
date: 0001-01-01
summary: "转换处理器 #  convert 处理器将文档中的字段转换为不同的类型，例如，将字符串转换为整数或将整数转换为字符串。对于数组字段，数组中的所有值都会被转换。
语法 #  以下是为 convert 处理器提供的语法：
{ &#34;convert&#34;: { &#34;field&#34;: &#34;field_name&#34;, &#34;type&#34;: &#34;type-value&#34; } } 配置参数 #  下表列出了 convert 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要转换的数据的字段名称。支持模板使用。   type 必填 要将字段值转换为的类型。支持的类型有 integer 、 long 、 float 、 double 、 string 、 boolean 、 ip 和 auto 。如果 type 是 boolean ，则如果字段值是字符串 true （忽略大小写），则将其设置为 true ；如果字段值是字符串 false （忽略大小写），则将其设置为 false 。如果类型设置为 ip ，则此处理器将验证字段值是否遵循 IPv4 或 IPv6 地址的正确格式；如果值无效，将引发错误。如果值不是允许的值之一，将发生错误。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。   target_field 可选 字段名称，用于存储解析后的数据。如果未指定，值将存储在 field 字段中。默认为 field 。    如何使用 #  按照以下步骤在管道中使用处理器。"
---



# 转换处理器

`convert` 处理器将文档中的字段转换为不同的类型，例如，将字符串转换为整数或将整数转换为字符串。对于数组字段，数组中的所有值都会被转换。


## 语法

以下是为 `convert` 处理器提供的语法：

```
{
    "convert": {
        "field": "field_name",
        "type": "type-value"
    }
}
```

## 配置参数

下表列出了 `convert` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `field`          | 必填     | 包含要转换的数据的字段名称。支持模板使用。                                                                                                                                                                                                                                                                                                                                                                                                 |
| `type`           | 必填     | 要将字段值转换为的类型。支持的类型有 `integer` 、 `long` 、 `float` 、 `double` 、 `string` 、 `boolean` 、 `ip` 和 `auto` 。如果 `type` 是 `boolean` ，则如果字段值是字符串 `true` （忽略大小写），则将其设置为 `true` ；如果字段值是字符串 `false` （忽略大小写），则将其设置为 `false` 。如果类型设置为 `ip` ，则此处理器将验证字段值是否遵循 IPv4 或 IPv6 地址的正确格式；如果值无效，将引发错误。如果值不是允许的值之一，将发生错误。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `if`             | 可选     | 处理器运行的条件。                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                                                                                                                                                                                                                                                                                                          |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。                                                                                                                                                                                                                                                                                                                  |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                                                                                                                                                                                                                                                                                                           |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                                                                                                                                                                                                                                                                                                     |
| `target_field`   | 可选     | 字段名称，用于存储解析后的数据。如果未指定，值将存储在 field 字段中。默认为 field 。                                                                                                                                                                                                                                                                                                                                                       |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `convert-price` 的管道，将 `price` 转换为浮点数，将转换后的值存储在 `price_float` 字段中，如果该值小于 0 ，则将其设置为 0 ：

```
PUT _ingest/pipeline/convert-price
{
  "description": "Pipeline that converts price to floating-point number and sets value to zero if price less than zero",
  "processors": [
    {
      "convert": {
        "field": "price",
        "type": "float",
        "target_field": "price_float"
      }
    },
    {
      "set": {
        "field": "price",
        "value": "0",
        "if": "ctx.price_float < 0"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/convert-price/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
       "_source": {
        "price": "-10.5"
      }
    }
  ]
}
```

返回内容确认管道按预期工作：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "price_float": -10.5,
          "price": "0"
        },
        "_ingest": {
          "timestamp": "2023-08-22T15:38:21.180688799Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=convert-price
{
  "price": "10.5"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```


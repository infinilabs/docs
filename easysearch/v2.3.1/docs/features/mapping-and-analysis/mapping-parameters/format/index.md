---
title: "格式参数（Format）"
date: 0001-01-01
summary: "Format 参数 #  format 参数指定日期字段的解析格式。Easysearch 内置了多种日期格式，也支持自定义格式字符串。
相关指南（先读这些） #    映射基础  Date 字段类型  基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;created_at&#34;: { &#34;type&#34;: &#34;date&#34;, &#34;format&#34;: &#34;yyyy-MM-dd HH:mm:ss&#34; } } } } 多格式支持 #  使用 || 分隔符可以指定多个格式，Easysearch 会依次尝试每种格式进行解析：
&#34;created_at&#34;: { &#34;type&#34;: &#34;date&#34;, &#34;format&#34;: &#34;yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis&#34; } 内置日期格式 #     格式 说明 示例     epoch_millis 毫秒时间戳 1618000000000   epoch_second 秒时间戳 1618000000   date_optional_time / strict_date_optional_time ISO 8601 日期，时间部分可选 2021-04-10T08:00:00Z   basic_date yyyyMMdd 20210410   basic_date_time yyyyMMdd&rsquo;T&rsquo;HHmmss."
---


# Format 参数

`format` 参数指定日期字段的解析格式。Easysearch 内置了多种日期格式，也支持自定义格式字符串。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [Date 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/date-field-type/" >}})

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}
```

## 多格式支持

使用 `||` 分隔符可以指定多个格式，Easysearch 会依次尝试每种格式进行解析：

```json
"created_at": {
  "type": "date",
  "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
}
```

## 内置日期格式

| 格式 | 说明 | 示例 |
|------|------|------|
| `epoch_millis` | 毫秒时间戳 | `1618000000000` |
| `epoch_second` | 秒时间戳 | `1618000000` |
| `date_optional_time` / `strict_date_optional_time` | ISO 8601 日期，时间部分可选 | `2021-04-10T08:00:00Z` |
| `basic_date` | yyyyMMdd | `20210410` |
| `basic_date_time` | yyyyMMdd'T'HHmmss.SSSZ | `20210410T080000.000Z` |
| `date` | yyyy-MM-dd | `2021-04-10` |
| `date_hour_minute_second` | yyyy-MM-dd'T'HH:mm:ss | `2021-04-10T08:00:00` |
| `date_time` | yyyy-MM-dd'T'HH:mm:ss.SSSZ | `2021-04-10T08:00:00.000Z` |
| `hour_minute_second` | HH:mm:ss | `08:00:00` |

> 完整的内置格式列表请参考 [Date 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/date-field-type/" >}})。

## 自定义格式字符串

| 符号 | 含义 | 示例 |
|------|------|------|
| `yyyy` | 四位年份 | 2021 |
| `MM` | 两位月份 | 04 |
| `dd` | 两位日期 | 10 |
| `HH` | 24 小时制小时 | 08 |
| `mm` | 分钟 | 30 |
| `ss` | 秒 | 45 |
| `SSS` | 毫秒 | 123 |
| `Z` | 时区偏移 | +0800 |

## 默认格式

如果不指定 `format`，日期字段默认使用 `strict_date_optional_time||epoch_millis`，这意味着它接受：

- ISO 8601 格式的日期/日期时间字符串
- 毫秒时间戳

## 注意事项

- `format` 参数在索引创建后**无法修改**，需要重建索引
- 写入的日期值必须匹配指定的某一种格式，否则会报解析错误
- 如果需要同时接受多种格式，使用 `||` 分隔
- `strict_` 前缀表示严格模式，不允许省略前导零（如月份 `4` 必须写为 `04`）


---
title: "支持的单位清单"
date: 0001-01-01
summary: "支持的单位清单 #  Easysearch 在 REST API 中支持以下几类单位，适用于查询、索引设置和集群配置。
时间单位 #     缩写 含义 示例     d 天 5d   h 小时 12h   m 分钟 30m   s 秒 60s   ms 毫秒 500ms   micros 微秒 100micros   nanos 纳秒 1000nanos    常见使用场景：timeout、scroll、interval、ILM 策略中的 min_age 等。
字节大小单位 #     缩写 含义 换算     b 字节 1 B   kb Kibibyte 1024 B   mb Mebibyte 1024 KB   gb Gibibyte 1024 MB   tb Tebibyte 1024 GB   pb Pebibyte 1024 TB     注意：虽然缩写看起来是十进制（kb），但实际按二进制计算（1 kb = 1024 bytes）。"
---


# 支持的单位清单

Easysearch 在 REST API 中支持以下几类单位，适用于查询、索引设置和集群配置。

## 时间单位

| 缩写     | 含义   | 示例        |
| :------- | :----- | :---------- |
| `d`      | 天     | `5d`        |
| `h`      | 小时   | `12h`       |
| `m`      | 分钟   | `30m`       |
| `s`      | 秒     | `60s`       |
| `ms`     | 毫秒   | `500ms`     |
| `micros` | 微秒   | `100micros` |
| `nanos`  | 纳秒   | `1000nanos` |

常见使用场景：`timeout`、`scroll`、`interval`、ILM 策略中的 `min_age` 等。

## 字节大小单位

| 缩写 | 含义       | 换算     |
| :--- | :--------- | :------- |
| `b`  | 字节       | 1 B      |
| `kb` | Kibibyte   | 1024 B   |
| `mb` | Mebibyte   | 1024 KB  |
| `gb` | Gibibyte   | 1024 MB  |
| `tb` | Tebibyte   | 1024 GB  |
| `pb` | Pebibyte   | 1024 TB  |

> **注意**：虽然缩写看起来是十进制（kb），但实际按二进制计算（1 kb = 1024 bytes）。

## 距离单位

| 缩写       | 含义 |
| :--------- | :--- |
| `mi`       | 英里 |
| `yd`       | 码   |
| `ft`       | 英尺 |
| `in`       | 英寸 |
| `km`       | 千米 |
| `m`        | 米   |
| `cm`       | 厘米 |
| `mm`       | 毫米 |
| `nmi`/`NM` | 海里 |

常见使用场景：`geo_distance` 查询和聚合、`geo_bounding_box` 过滤等。

## 无单位数量

| 缩写 | 含义   | 换算          |
| :--- | :----- | :------------ |
| `k`  | 千     | 1,000         |
| `m`  | 百万   | 1,000,000     |
| `g`  | 十亿   | 1,000,000,000 |
| `t`  | 万亿   | 10^12         |
| `p`  | 千万亿 | 10^15         |


要将 API 响应中的数值自动转换为人类可读格式，请参阅[常用 REST 参数]({{< relref "./common-parameters.md" >}})。


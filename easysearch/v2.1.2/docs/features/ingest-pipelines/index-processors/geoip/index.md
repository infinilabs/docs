---
title: "GeoIP 处理器"
date: 0001-01-01
summary: "GeoIP 处理器 #   需要 ingest-geoip 插件
 geoip 处理器根据 IP 地址查询 MaxMind GeoLite2 数据库，为文档添加地理位置信息，包括国家、城市、经纬度、时区和 ASN 等。该处理器非常适合日志分析、流量审计和用户来源可视化等场景。
语法 #  { &#34;geoip&#34;: { &#34;field&#34;: &#34;ip&#34; } } 配置参数 #     参数 是否必填 描述     field 必填 包含 IP 地址的源字段名。支持 IPv4 和 IPv6   target_field 可选 存储地理位置信息的目标字段。默认为 geoip   database_file 可选 使用的 MaxMind 数据库文件名。默认为 GeoLite2-City.mmdb   properties 可选 要提取的属性列表。未指定时根据数据库类型使用默认属性集（见下表）   ignore_missing 可选 为 true 时，如果源字段缺失或为 null，则跳过处理不报错。默认为 false   first_only 可选 当源字段为数组时，为 true 仅返回第一个匹配结果；为 false 返回所有匹配结果的列表。默认为 true   description 可选 处理器的简要描述   if 可选 处理器运行的条件   ignore_failure 可选 为 true 时，处理器出错后忽略继续执行。默认为 false   on_failure 可选 处理器失败时运行的处理器列表   tag 可选 处理器的标识标签    支持的数据库 #  处理器根据数据库类型后缀自动识别数据库格式："
---


# GeoIP 处理器

> **需要 ingest-geoip 插件**

`geoip` 处理器根据 IP 地址查询 MaxMind GeoLite2 数据库，为文档添加地理位置信息，包括国家、城市、经纬度、时区和 ASN 等。该处理器非常适合日志分析、流量审计和用户来源可视化等场景。

## 语法

```json
{
  "geoip": {
    "field": "ip"
  }
}
```

## 配置参数

| 参数 | 是否必填 | 描述 |
|------|---------|------|
| `field` | 必填 | 包含 IP 地址的源字段名。支持 IPv4 和 IPv6 |
| `target_field` | 可选 | 存储地理位置信息的目标字段。默认为 `geoip` |
| `database_file` | 可选 | 使用的 MaxMind 数据库文件名。默认为 `GeoLite2-City.mmdb` |
| `properties` | 可选 | 要提取的属性列表。未指定时根据数据库类型使用默认属性集（见下表） |
| `ignore_missing` | 可选 | 为 `true` 时，如果源字段缺失或为 null，则跳过处理不报错。默认为 `false` |
| `first_only` | 可选 | 当源字段为数组时，为 `true` 仅返回第一个匹配结果；为 `false` 返回所有匹配结果的列表。默认为 `true` |
| `description` | 可选 | 处理器的简要描述 |
| `if` | 可选 | 处理器运行的条件 |
| `ignore_failure` | 可选 | 为 `true` 时，处理器出错后忽略继续执行。默认为 `false` |
| `on_failure` | 可选 | 处理器失败时运行的处理器列表 |
| `tag` | 可选 | 处理器的标识标签 |

## 支持的数据库

处理器根据数据库类型后缀自动识别数据库格式：

| 数据库 | 文件名 | 说明 |
|--------|--------|------|
| GeoLite2-City | `GeoLite2-City.mmdb` | 城市级定位，含经纬度（**默认**） |
| GeoLite2-Country | `GeoLite2-Country.mmdb` | 国家级定位 |
| GeoLite2-ASN | `GeoLite2-ASN.mmdb` | 自治系统编号（ASN）信息 |

也可以将自定义的 `.mmdb` 数据库文件放入 `plugins/ingest-geoip/` 目录使用。

## 可提取属性

### City 数据库（默认）

| 属性 | 说明 | 默认提取 |
|------|------|---------|
| `continent_name` | 所在大洲 | ✅ |
| `country_iso_code` | ISO 国家代码 | ✅ |
| `country_name` | 国家名称 | ✅ |
| `region_iso_code` | ISO 3166-2 地区代码（如 `CN-BJ`） | ✅ |
| `region_name` | 地区/省份名称 | ✅ |
| `city_name` | 城市名称 | ✅ |
| `location` | 经纬度坐标（`{"lat": ..., "lon": ...}`） | ✅ |
| `timezone` | 时区 | ❌ |
| `ip` | 格式化的 IP 地址 | ❌ |

### Country 数据库

| 属性 | 说明 | 默认提取 |
|------|------|---------|
| `continent_name` | 所在大洲 | ✅ |
| `country_iso_code` | ISO 国家代码 | ✅ |
| `country_name` | 国家名称 | ✅ |
| `ip` | 格式化的 IP 地址 | ❌ |

### ASN 数据库

| 属性 | 说明 | 默认提取 |
|------|------|---------|
| `ip` | 格式化的 IP 地址 | ✅ |
| `asn` | 自治系统编号 | ✅ |
| `organization_name` | 组织名称 | ✅ |
| `network` | 网段 CIDR | ✅ |

## 如何使用

### 步骤 1：安装插件

```bash
bin/easysearch-plugin install ingest-geoip
```

安装后重启节点。

### 步骤 2：创建管道

以下管道使用 City 数据库为包含 IP 地址的文档添加地理位置信息：

```json
PUT /_ingest/pipeline/geoip_pipeline
{
  "description": "根据 IP 地址添加地理位置信息",
  "processors": [
    {
      "geoip": {
        "field": "client_ip"
      }
    }
  ]
}
```

### 步骤 3：测试管道

```json
POST /_ingest/pipeline/geoip_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "client_ip": "8.8.8.8"
      }
    }
  ]
}
```

**响应示例**：

```json
{
  "docs": [
    {
      "doc": {
        "_source": {
          "client_ip": "8.8.8.8",
          "geoip": {
            "continent_name": "North America",
            "country_iso_code": "US",
            "country_name": "United States",
            "region_iso_code": "US-CA",
            "region_name": "California",
            "city_name": "Mountain View",
            "location": {
              "lat": 37.386,
              "lon": -122.0838
            }
          }
        }
      }
    }
  ]
}
```

### 使用 ASN 数据库

```json
PUT /_ingest/pipeline/geoip_asn_pipeline
{
  "description": "根据 IP 地址添加 ASN 信息",
  "processors": [
    {
      "geoip": {
        "field": "client_ip",
        "target_field": "asn_info",
        "database_file": "GeoLite2-ASN.mmdb"
      }
    }
  ]
}
```

### 选择特定属性

只提取国家代码和城市名称：

```json
{
  "geoip": {
    "field": "client_ip",
    "properties": ["country_iso_code", "city_name"]
  }
}
```

### 结合多个数据库

同时获取城市定位和 ASN 信息：

```json
PUT /_ingest/pipeline/geoip_full_pipeline
{
  "description": "获取完整的地理位置和 ASN 信息",
  "processors": [
    {
      "geoip": {
        "field": "client_ip",
        "target_field": "geo"
      }
    },
    {
      "geoip": {
        "field": "client_ip",
        "target_field": "asn",
        "database_file": "GeoLite2-ASN.mmdb"
      }
    }
  ]
}
```

## 注意事项

- 如果 IP 地址在数据库中未找到，处理器不会添加 `target_field`（不会报错）
- 查询结果会缓存在内存中（默认缓存 1000 条），可通过节点设置 `ingest.geoip.cache_size` 调整缓存大小
- 数据库默认通过内存映射文件（mmap）加载；设置系统属性 `es.geoip.load_db_on_heap=true` 可将数据库完整加载到堆内存


---
title: "聚合场景实践"
date: 0001-01-01
description: "按业务场景拆解的聚合实战示例：看板统计、时间序列分析与去重计数等。"
summary: "聚合场景实践 #  本页不再解释每种聚合的语法，而是按“业务问题”给出几类常见的聚合场景实践。
语法细节请先阅读前面的聚合基础/桶聚合/指标聚合章节。
场景一：按状态与应用统计错误率 #  需求：做一个简单的错误率看板，按应用和日志级别统计请求量与错误量。
思路：
 先用 bool.filter 限定时间窗口（如最近 15 分钟/1 小时） 使用 terms 按应用分桶 在每个应用桶内，再按日志级别分桶，并计算总数  示例：
GET /logs-*/_search { &#34;size&#34;: 0, &#34;query&#34;: { &#34;bool&#34;: { &#34;filter&#34;: [ { &#34;range&#34;: { &#34;@timestamp&#34;: { &#34;gte&#34;: &#34;now-15m&#34; } } } ] } }, &#34;aggs&#34;: { &#34;by_app&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;app_name.keyword&#34;, &#34;size&#34;: 20 }, &#34;aggs&#34;: { &#34;by_level&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;log_level.keyword&#34; } } } } } } 可以在应用层根据各级别计数计算错误率，或继续在子聚合中添加 filter 聚合专门统计错误级别。"
---


# 聚合场景实践

本页不再解释每种聚合的语法，而是按“业务问题”给出几类常见的聚合场景实践。  
语法细节请先阅读前面的聚合基础/桶聚合/指标聚合章节。

## 场景一：按状态与应用统计错误率

需求：做一个简单的错误率看板，按应用和日志级别统计请求量与错误量。

思路：

- 先用 `bool.filter` 限定时间窗口（如最近 15 分钟/1 小时）
- 使用 `terms` 按应用分桶
- 在每个应用桶内，再按日志级别分桶，并计算总数

示例：

```json
GET /logs-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-15m" } } }
      ]
    }
  },
  "aggs": {
    "by_app": {
      "terms": { "field": "app_name.keyword", "size": 20 },
      "aggs": {
        "by_level": {
          "terms": { "field": "log_level.keyword" }
        }
      }
    }
  }
}
```

可以在应用层根据各级别计数计算错误率，或继续在子聚合中添加 `filter` 聚合专门统计错误级别。

## 场景二：时间序列指标折线图（分服务）

需求：在监控看板上画出每个服务的 QPS 或平均延迟随时间变化的曲线。

思路：

- 外层用 `date_histogram` 按时间切分（如 1 分钟/5 分钟）
- 内层用 `terms` 按服务名分桶
- 在每个子桶内计算 `sum` 或 `avg`

示例（按服务统计平均延迟）：

```json
GET /apm-metrics-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  },
  "aggs": {
    "per_minute": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "1m"
      },
      "aggs": {
        "by_service": {
          "terms": { "field": "service.name.keyword", "size": 10 },
          "aggs": {
            "avg_latency": {
              "avg": { "field": "latency_ms" }
            }
          }
        }
      }
    }
  }
}
```

## 场景三：价格分布与直方图

需求：电商类场景中，查看某类商品的价格分布情况（例如画价格分布条形图）。

思路：

- 使用 `histogram` 或 `range` 聚合按价格分桶
- 在每个价格桶内统计文档数量

示例：

```json
GET /products/_search
{
  "size": 0,
  "query": {
    "term": { "category.keyword": "shoes" }
  },
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 300 },
          { "from": 300, "to": 1000 },
          { "from": 1000 }
        ]
      }
    }
  }
}
```

可以根据实际业务调整区间划分粒度。

## 场景四：UV/去重计数（cardinality）

需求：统计活跃用户数、访问 IP 数等，需要对某个字段做去重计数。

思路：

- 使用 `cardinality` 指标聚合对用户 ID 或 IP 字段做近似去重计数
- 可以结合 `date_histogram` 做时间序列 UV 曲线

示例（最近 1 天 UV）：

```json
GET /web-logs-*/_search
{
  "size": 0,
  "query": {
    "range": {
      "@timestamp": { "gte": "now-1d" }
    }
  },
  "aggs": {
    "uv": {
      "cardinality": {
        "field": "user_id.keyword",
        "precision_threshold": 40000
      }
    }
  }
}
```

`precision_threshold` 越大，精度越高但内存占用越多。

## 场景五：TopN 排行榜（terms + 排序）

需求：统计一段时间内访问量最高的页面、下单量最高的商品等。

思路：

- 使用 `terms` 聚合按对象 ID 或名称分桶
- 通过 `order` 按子聚合（如 `sum` 或 `count`）排序

示例（下单量 TopN 商品）：

```json
GET /orders-*/_search
{
  "size": 0,
  "query": {
    "range": {
      "order_time": { "gte": "now-7d" }
    }
  },
  "aggs": {
    "top_products": {
      "terms": {
        "field": "product_id",
        "size": 20,
        "order": { "total_qty": "desc" }
      },
      "aggs": {
        "total_qty": {
          "sum": { "field": "quantity" }
        }
      }
    }
  }
}
```

## 场景六：结合 Geo 的热点分析与附近搜索

Geo 更详细的配方在 [地理位置场景实践]({{< relref "/docs/features/geo-search/geo-recipes.md" >}}) 中，这里挑几个和聚合结合较多的典型场景。

### 6.1 门店热力图（geohash_grid）

需求：在地图上展示“哪里门店/打车点最密集”，并按区域统计单量或客单价。

思路：

- 使用 `geohash_grid` 或 `geotile_grid` 聚合，将坐标点聚合到网格；
- 在每个网格桶内再计算业务指标（如门店数、订单数、平均价格）。

示例：

```json
GET /stores/_search
{
  "size": 0,
  "aggs": {
    "grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 5
      },
      "aggs": {
        "store_count": { "value_count": { "field": "id" } },
        "avg_order_price": { "avg": { "field": "avg_order_price" } }
      }
    }
  }
}
```

前端可以基于 `grid` 的 buckets 绘制热力图或气泡图。

### 6.2 “附近 N 公里”内的指标（geo_distance + 聚合）

需求：以用户当前位置为圆心，统计 **3km / 5km / 10km** 内的门店数量、平均评分等。

思路：

- 查询阶段用 `geo_distance` 过滤一个最大半径（如 10km）；
- 聚合阶段用 `geo_distance` 桶聚合划分距离区间，在每个距离桶内再做指标聚合。

示例：

```json
GET /stores/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        {
          "geo_distance": {
            "distance": "10km",
            "location": {
              "lat": 31.2304,
              "lon": 121.4737
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "by_distance": {
      "geo_distance": {
        "field": "location",
        "origin": { "lat": 31.2304, "lon": 121.4737 },
        "unit": "km",
        "ranges": [
          { "to": 3 },
          { "from": 3, "to": 5 },
          { "from": 5, "to": 10 }
        ]
      },
      "aggs": {
        "store_count": { "value_count": { "field": "id" } },
        "avg_rating":  { "avg": { "field": "rating" } }
      }
    }
  }
}
```

### 6.3 按时间 + 地理维度做统计（date_histogram + geohash_grid）

需求：查看最近 7 天内，不同区域的订单量变化趋势（例如画“时间 × 区域”的热力矩阵）。

思路：

- 外层 `date_histogram` 按天切时间；
- 内层 `geohash_grid`（或按行政区域 terms）分地理区域；
- 在每个子桶内统计订单笔数或 GMV。

示例（时间 × geohash 网格的订单量）：

```json
GET /orders-*/_search
{
  "size": 0,
  "query": {
    "range": {
      "order_time": { "gte": "now-7d" }
    }
  },
  "aggs": {
    "per_day": {
      "date_histogram": {
        "field": "order_time",
        "calendar_interval": "1d"
      },
      "aggs": {
        "by_area": {
          "geohash_grid": {
            "field": "store_location",
            "precision": 4
          },
          "aggs": {
            "order_count": { "value_count": { "field": "order_id" } }
          }
        }
      }
    }
  }
}
```

这类结果可以在前端渲染成“按天 × 区域”的二维矩阵，帮助定位某些时间段、某些区域的异常波动。

## 参考手册（API 与参数）

- [聚合概览（功能手册）]({{< relref "/docs/features/aggregations/" >}})
- [桶聚合（功能手册）]({{< relref "/docs/features/aggregations/bucket-aggregations/" >}})
- [指标聚合（功能手册）]({{< relref "/docs/features/aggregations/metric-aggregations/" >}})
- [管道聚合（功能手册）]({{< relref "/docs/features/aggregations/pipeline-aggregations/" >}})


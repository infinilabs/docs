---
title: "指标聚合"
date: 0001-01-01
summary: "📖 概念与教程请阅读 指标聚合教程
 指标聚合 - API 参考 #  本节详细列出所有指标聚合的参数与用法。
相关资源 #    聚合基础教程  指标聚合教程（概念、最佳实践、常见用法）  聚合场景实践   指标聚合类型 #  指标聚合分为单值聚合（返回单个数值）和多值聚合（返回多个数值）两类。
单值指标聚合 #  返回单个数值的指标，适合基础统计：
   聚合 说明 常见用途      avg 计算平均值 平均延迟、平均收入    sum 计算总和 总销售额、总请求数    min 计算最小值 最小延迟、最低价格    max 计算最大值 最大延迟、最高价格    cardinality 去重计数（近似值） 独立用户数、不同IP数    value_count 计算字段值数量 有值的文档数    median_absolute_deviation 中位数绝对偏差 分布离散度分析    weighted_avg 加权平均值 加权平均评分、加权成本    rate 速率计算（单位时间内的值） 每日请求速率、年化收入    多值指标聚合 #  返回多个数值的指标，一次获取多种统计："
---


> 📖 **概念与教程请阅读** [指标聚合教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})

# 指标聚合 - API 参考

本节详细列出所有指标聚合的参数与用法。

## 相关资源

- [聚合基础教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [指标聚合教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})（概念、最佳实践、常见用法）
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

---

## 指标聚合类型

指标聚合分为**单值聚合**（返回单个数值）和**多值聚合**（返回多个数值）两类。

### 单值指标聚合

返回单个数值的指标，适合基础统计：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [avg]({{< relref "average.md" >}}) | 计算平均值 | 平均延迟、平均收入 |
| [sum]({{< relref "sum.md" >}}) | 计算总和 | 总销售额、总请求数 |
| [min]({{< relref "minimum.md" >}}) | 计算最小值 | 最小延迟、最低价格 |
| [max]({{< relref "maximum.md" >}}) | 计算最大值 | 最大延迟、最高价格 |
| [cardinality]({{< relref "cardinality.md" >}}) | 去重计数（近似值） | 独立用户数、不同IP数 |
| [value_count]({{< relref "value-count.md" >}}) | 计算字段值数量 | 有值的文档数 |
| [median_absolute_deviation]({{< relref "median-absolute-deviation.md" >}}) | 中位数绝对偏差 | 分布离散度分析 |
| [weighted_avg]({{< relref "weighted-avg.md" >}}) | 加权平均值 | 加权平均评分、加权成本 |
| [rate]({{< relref "rate.md" >}}) | 速率计算（单位时间内的值） | 每日请求速率、年化收入 |

### 多值指标聚合

返回多个数值的指标，一次获取多种统计：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [stats]({{< relref "stats.md" >}}) | 返回 count、min、max、avg、sum | 一次获取基本统计信息 |
| [extended_stats]({{< relref "extended-stats.md" >}}) | stats + 方差、标准差、四分位等 | 详细的分布统计 |
| [percentiles]({{< relref "percentile.md" >}}) | 百分位数（P50、P95、P99 等） | 性能分析、SLA监控 |
| [percentile_ranks]({{< relref "percentile-ranks.md" >}}) | 值所在的百分位 | 给定值的排名分析 |
| [top_hits]({{< relref "top-hits.md" >}}) | 返回桶内 Top N 文档 | 查看每组的示例记录 |
| [scripted_metric]({{< relref "scripted-metric.md" >}}) | 使用脚本自定义指标计算 | 复杂的自定义统计 |

### 地理位置指标

用于地理数据的统计：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [geo_bounds]({{< relref "geobounds.md" >}}) | 地理点的边界框 | 地图缩放范围 |
| [geo_centroid]({{< relref "geocentroid.md" >}}) | 地理点的中心点 | 地图中心位置 |

### 矩阵指标

跨多个字段的联合统计：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [matrix_stats]({{< relref "matrix-stats.md" >}}) | 多字段的相关性矩阵统计 | 字段间相关性分析、多变量统计 |

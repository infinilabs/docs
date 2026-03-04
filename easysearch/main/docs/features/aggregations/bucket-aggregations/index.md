---
title: "桶聚合"
date: 0001-01-01
summary: "📖 概念与教程请阅读 桶聚合教程
 桶聚合 - API 参考 #  本节详细列出所有桶聚合的参数与用法。
相关资源 #    聚合基础教程  桶聚合教程（概念、最佳实践、常见用法）  聚合场景实践   桶聚合类型 #  词项聚合 (4 种) #  按字段值分组，适合离散维度：
   聚合 说明 常见用途      terms 按字段值分桶，返回 Top N 词项 按状态、地区、应用分组；热词统计    rare_terms 找出低频长尾词项 异常值检测；稀有事件分析    significant_terms 统计显著性词项（相对背景集） 重点词项提取；对比分析    significant_text 统计显著性文本（支持 text 字段） text 字段的显著词分析    数值分桶 (3 种) #  按数值范围分组："
---


> 📖 **概念与教程请阅读** [桶聚合教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})

# 桶聚合 - API 参考

本节详细列出所有桶聚合的参数与用法。

## 相关资源

- [聚合基础教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [桶聚合教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})（概念、最佳实践、常见用法）
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

---

## 桶聚合类型

### 词项聚合 (4 种)

按字段值分组，适合离散维度：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [terms]({{< relref "terms.md" >}}) | 按字段值分桶，返回 Top N 词项 | 按状态、地区、应用分组；热词统计 |
| [rare_terms]({{< relref "rare-terms.md" >}}) | 找出低频长尾词项 | 异常值检测；稀有事件分析 |
| [significant_terms]({{< relref "significant-terms.md" >}}) | 统计显著性词项（相对背景集） | 重点词项提取；对比分析 |
| [significant_text]({{< relref "significant-text.md" >}}) | 统计显著性文本（支持 text 字段） | text 字段的显著词分析 |

### 数值分桶 (3 种)

按数值范围分组：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [histogram]({{< relref "histogram.md" >}}) | 数值等距分桶 | 价格分布、评分分布、年龄分布 |
| [variable_width_histogram]({{< relref "variable-width-histogram.md" >}}) | 自适应宽度分桶 | 数据分布不均匀时的自动分桶 |
| [range]({{< relref "range.md" >}}) | 自定义数值区间分桶 | 自定义价格段、年龄段、等级划分 |

### 时间分桶 (3 种)

按时间单位分组，用于时间序列分析：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [date_histogram]({{< relref "date-histogram.md" >}}) | 按日期/时间粒度分桶 | 趋势分析；时间序列数据 |
| [date_range]({{< relref "date-range.md" >}}) | 自定义日期区间分桶 | 自定义时间段分析；历史对比 |
| [auto_date_histogram]({{< relref "auto-interval-date-histogram.md" >}}) | 自动选择最佳时间粒度 | 自适应时间聚合 |

### 地理位置分桶 (3 种)

按地理位置分组：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [geo_distance]({{< relref "geo-distance.md" >}}) | 按距离某点的距离分桶 | 附近搜索；距离分析 |
| [geohash_grid]({{< relref "geohash-grid.md" >}}) | 按 Geohash 网格分桶 | 地图热力图；空间聚类 |
| [geotile_grid]({{< relref "geotile-grid.md" >}}) | 按 Web 地图瓦片分桶 | 地图应用；Web 地图可视化 |

### IP 地址分桶 (1 种)

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [ip_range]({{< relref "ip-range.md" >}}) | 按 IP 地址区间分桶 | IP 地址段分析；网络统计 |

### 嵌套与关联 (4 种)

用于复杂数据结构：

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [nested]({{< relref "nested.md" >}}) | 对 nested 对象做聚合 | 评论聚合、订单项聚合、嵌套数据分析 |
| [reverse_nested]({{< relref "reverse-nested.md" >}}) | 从 nested 回到 root 文档 | 嵌套数据上的父级聚合 |
| [children]({{< relref "children.md" >}}) | 按父子关系向下聚合 | 父子关系数据的子集分析 |
| [parent]({{< relref "parent.md" >}}) | 按父子关系向上聚合 | 子文档的父级信息聚合 |

### 过滤与采样 (4 种)

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [filter]({{< relref "filter.md" >}}) | 单条件过滤桶 | 条件分组；子集统计 |
| [filters]({{< relref "filters.md" >}}) | 多条件过滤桶 | 多维条件分组；多方案对比 |
| [sampler]({{< relref "sampler.md" >}}) | 采样聚合，减少数据量 | 大数据集采样分析；性能优化 |
| [diversified_sampler]({{< relref "diversified-sampler.md" >}}) | 多样化采样 | 多样本采样；均衡采样 |

### 特殊桶聚合 (4 种)

| 聚合 | 说明 | 常见用途 |
|------|------|----------|
| [composite]({{< relref "composite.md" >}}) | 组合分桶，支持分页遍历所有桶 | 大量桶的分页遍历；结果流式处理 |
| [adjacency_matrix]({{< relref "adjacency-matrix.md" >}}) | 多过滤条件交集矩阵 | 条件组合分析；热力矩阵 |
| [missing]({{< relref "missing.md" >}}) | 字段缺失值桶 | 数据质量分析；缺失值统计 |
| [global]({{< relref "global.md" >}}) | 跳过 query 约束的全局桶 | 全局统计；对比过滤结果 |

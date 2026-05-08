---
title: "映射参数"
date: 0001-01-01
description: "字段映射参数详解，控制字段的索引、存储和搜索行为"
summary: "映射参数 #  映射参数用于控制字段的索引方式、存储行为和搜索特性。合理配置这些参数可以优化查询性能和存储效率。
 概念指南 → 了解如何设计映射，请阅读：
  映射基础  映射模式与常见坑    按功能分类 #  索引控制 #  控制字段是否可被搜索以及如何索引。
   参数 说明 默认值      index 是否索引该字段（可搜索） true    enabled 是否解析和索引该对象字段 true    doc_values 是否存储列式数据（用于排序/聚合） true    store 是否独立存储字段值 false    norms 是否存储长度归一化因子 text: true    文本分析 #  控制文本字段的分词和搜索行为。
   参数 说明 默认值      analyzer 索引和搜索时的分词器 standard    search_analyzer 仅搜索时使用的分词器 同 analyzer    search_quote_analyzer 带引号短语查询时使用的分词器 同 search_analyzer    normalizer keyword 字段的归一化器 无    term_vector 是否存储词条向量 no    position_increment_gap 数组元素间的位置间隔 100    fielddata_frequency_filter 字段数据频率过滤器配置 无    index_phrases 是否为短语查询构建额外索引 false    index_prefixes 是否为前缀查询构建额外索引 false    split_queries_on_whitespace 查询是否在空白处拆分 false    数据转换 #  控制数据类型转换和格式化。"
---


# 映射参数

映射参数用于控制字段的索引方式、存储行为和搜索特性。合理配置这些参数可以优化查询性能和存储效率。

> **概念指南** → 了解如何设计映射，请阅读：
> - [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
> - [映射模式与常见坑]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

---

## 按功能分类

### 索引控制

控制字段是否可被搜索以及如何索引。

| 参数 | 说明 | 默认值 |
|------|------|--------|
| [index](index.md) | 是否索引该字段（可搜索） | `true` |
| [enabled](enabled.md) | 是否解析和索引该对象字段 | `true` |
| [doc_values](doc_values.md) | 是否存储列式数据（用于排序/聚合） | `true` |
| [store](store.md) | 是否独立存储字段值 | `false` |
| [norms](norms.md) | 是否存储长度归一化因子 | text: `true` |

### 文本分析

控制文本字段的分词和搜索行为。

| 参数 | 说明 | 默认值 |
|------|------|--------|
| [analyzer](analyzer.md) | 索引和搜索时的分词器 | `standard` |
| [search_analyzer](search_analyzer.md) | 仅搜索时使用的分词器 | 同 `analyzer` |
| [search_quote_analyzer](search_quote_analyzer.md) | 带引号短语查询时使用的分词器 | 同 `search_analyzer` |
| [normalizer](normalizer.md) | keyword 字段的归一化器 | 无 |
| [term_vector](term_vector.md) | 是否存储词条向量 | `no` |
| [position_increment_gap](position_increment_gap.md) | 数组元素间的位置间隔 | `100` |
| [fielddata_frequency_filter](fielddata_frequency_filter.md) | 字段数据频率过滤器配置 | 无 |
| [index_phrases](index_phrases.md) | 是否为短语查询构建额外索引 | `false` |
| [index_prefixes](index_prefixes.md) | 是否为前缀查询构建额外索引 | `false` |
| [split_queries_on_whitespace](split_queries_on_whitespace.md) | 查询是否在空白处拆分 | `false` |

### 数据转换

控制数据类型转换和格式化。

| 参数 | 说明 | 默认值 |
|------|------|--------|
| [format](format.md) | 日期字段的解析格式 | `strict_date_optional_time\|\|epoch_millis` |
| [locale](locale.md) | 解析日期/范围时使用的区域设置 | 无 |
| [coerce](coerce.md) | 是否自动转换数据类型 | `true` |
| [null_value](null_value.md) | null 的替代值 | 无 |
| [ignore_malformed](ignore_malformed.md) | 是否忽略格式错误的值 | `false` |
| [ignore_above](ignore_above.md) | 超长字符串不索引的阈值 | `2147483647` |
| [ignore_z_value](ignore_z_value.md) | 是否忽略地理坐标的 Z 值 | `true` |

### 字段结构

控制字段的组织和扩展方式。

| 参数 | 说明 | 默认值 |
|------|------|--------|
| [fields](fields.md) | 多字段定义（同字段多种索引方式） | 无 |
| [properties](properties.md) | 对象/嵌套字段的子字段定义 | 无 |
| [copy_to](copy_to.md) | 将字段值复制到目标字段 | 无 |
| [dynamic](dynamic.md) | 是否动态添加新字段 | `true` |

### 搜索与评分

控制搜索行为和相关性评分。

| 参数 | 说明 | 默认值 |
|------|------|--------|
| [boost](boost.md) | 字段权重提升因子 | `1.0` |
| [similarity](similarity.md) | 相关性评分算法 | `BM25` |
| [fielddata](fielddata.md) | text 字段是否启用堆内存聚合 | `false` |
| [eager_global_ordinals](eager_global_ordinals.md) | 是否提前构建全局序数 | `false` |

### 完成字段（Completion）

特定于自动完成/搜索建议的参数。

| 参数 | 说明 | 默认值 |
|------|------|--------|
| [preserve_separators](preserve_separators.md) | 是否保留分隔符 | `true` |
| [preserve_position_increments](preserve_position_increments.md) | 是否保留位置增量 | `true` |
| [contexts](contexts.md) | 完成上下文定义 | 无 |
| [max_input_length](max_input_length.md) | 最大输入长度 | `50` |

### Mapper 扩展模块参数

mapper-extras 模块提供的特殊字段类型的参数。

| 参数 | 字段类型 | 说明 | 默认值 |
|------|---------|------|--------|
| [scaling_factor](scaling_factor.md) | `scaled_float` | 浮点数的缩放因子 | — |
| [positive_score_impact](positive_score_impact.md) | `rank_feature` | 特性对得分的影响 | `true` |
| [enable_position_increments](enable_position_increments.md) | `token_count` | 是否包含位置增量 | `true` |
| [max_shingle_size](max_shingle_size.md) | `search_as_you_type` | 最大 shingle 大小 | `3` |

### 元数据

附加信息和说明。

| 参数 | 说明 | 默认值 |
|------|------|--------|
| [meta](meta.md) | 自定义元数据（单位、说明等） | 无 |

---

## 常用参数详解

### index — 是否可搜索

```json
{
  "properties": {
    "internal_id": {
      "type": "keyword",
      "index": false
    }
  }
}
```

- `true`（默认）：字段可被搜索
- `false`：字段不可搜索，但仍可聚合（如果 doc_values=true）

**使用场景**：仅用于聚合或排序的字段可设为 `index: false`

### doc_values — 列式存储

```json
{
  "properties": {
    "description": {
      "type": "text",
      "doc_values": false
    }
  }
}
```

- `true`（默认）：支持排序和聚合
- `false`：节省磁盘空间，但无法排序/聚合

**使用场景**：纯全文搜索字段可关闭 doc_values

### copy_to — 字段合并

```json
{
  "properties": {
    "title": {
      "type": "text",
      "copy_to": "full_content"
    },
    "body": {
      "type": "text",
      "copy_to": "full_content"
    },
    "full_content": {
      "type": "text"
    }
  }
}
```

将多个字段内容复制到一个目标字段，便于跨字段搜索。

### fields — 多字段

```json
{
  "properties": {
    "title": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword"
        },
        "english": {
          "type": "text",
          "analyzer": "english"
        }
      }
    }
  }
}
```

同一字段以多种方式索引：
- `title` → 默认分词搜索
- `title.keyword` → 精确匹配、聚合
- `title.english` → 英语词干搜索

### ignore_above — 截断长字符串

```json
{
  "properties": {
    "tag": {
      "type": "keyword",
      "ignore_above": 256
    }
  }
}
```

超过 256 字符的值不被索引（但仍存储在 _source 中）。

---

## 参数适用类型

| 参数 | text | keyword | numeric | date | boolean | object | nested |
|------|:----:|:-------:|:-------:|:----:|:-------:|:------:|:------:|
| analyzer | ✓ | - | - | - | - | - | - |
| boost | ✓ | ✓ | ✓ | ✓ | ✓ | - | - |
| coerce | - | - | ✓ | ✓ | - | - | - |
| copy_to | ✓ | ✓ | ✓ | ✓ | ✓ | - | - |
| doc_values | - | ✓ | ✓ | ✓ | ✓ | - | - |
| dynamic | - | - | - | - | - | ✓ | ✓ |
| enabled | - | - | - | - | - | ✓ | - |
| fielddata | ✓ | - | - | - | - | - | - |
| fields | ✓ | ✓ | ✓ | ✓ | ✓ | - | - |
| format | - | - | - | ✓ | - | - | - |
| ignore_above | - | ✓ | - | - | - | - | - |
| ignore_malformed | - | - | ✓ | ✓ | - | - | - |
| index | ✓ | ✓ | ✓ | ✓ | ✓ | - | - |
| meta | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| normalizer | - | ✓ | - | - | - | - | - |
| norms | ✓ | ✓ | - | - | - | - | - |
| null_value | - | ✓ | ✓ | ✓ | ✓ | - | - |
| properties | - | - | - | - | - | ✓ | ✓ |
| search_analyzer | ✓ | - | - | - | - | - | - |
| similarity | ✓ | - | - | - | - | - | - |
| store | ✓ | ✓ | ✓ | ✓ | ✓ | - | - |
| term_vector | ✓ | - | - | - | - | - | - |

---

## 完整参数列表

| 参数 | 描述 | 默认值 |
|------|------|--------|
| [analyzer](analyzer.md) | 指定索引和搜索时使用的分词器 | `standard` |
| [boost](boost.md) | 查询时的字段权重提升因子 | `1.0` |
| [coerce](coerce.md) | 控制是否自动将值转换为字段类型 | `true` |
| [contexts](contexts.md) | 完成字段的上下文定义 | — |
| [copy_to](copy_to.md) | 将字段值复制到目标字段 | — |
| [doc_values](doc_values.md) | 是否存储列式数据以加速排序和聚合 | `true` |
| [dynamic](dynamic.md) | 是否动态添加新字段 | `true` |
| [eager_global_ordinals](eager_global_ordinals.md) | 是否在刷新时提前构建全局序数 | `false` |
| [enable_position_increments](enable_position_increments.md) | 令牌计数是否包含位置增量 | `true` |
| [enabled](enabled.md) | 是否解析和索引该字段 | `true` |
| [fielddata](fielddata.md) | 是否为 text 字段启用堆内存排序/聚合 | `false` |
| [fielddata_frequency_filter](fielddata_frequency_filter.md) | 字段数据频率过滤器配置 | — |
| [fields](fields.md) | 为同一字段定义多种索引方式 | — |
| [format](format.md) | 指定日期字段的解析格式 | `strict_date_optional_time\|\|epoch_millis` |
| [ignore_above](ignore_above.md) | 超过指定长度的字符串不被索引 | `2147483647` |
| [ignore_malformed](ignore_malformed.md) | 是否忽略格式错误的值 | `false` |
| [ignore_z_value](ignore_z_value.md) | 是否忽略地理坐标的 Z 值 | `true` |
| [index](index.md) | 控制字段是否可被搜索 | `true` |
| [index_phrases](index_phrases.md) | 是否为短语查询构建额外索引 | `false` |
| [index_prefixes](index_prefixes.md) | 是否为前缀查询构建额外索引 | `false` |
| [locale](locale.md) | 解析日期/范围时的区域设置 | — |
| [max_input_length](max_input_length.md) | 完成字段的最大输入长度 | `50` |
| [max_shingle_size](max_shingle_size.md) | search_as_you_type 的最大 shingle 大小 | `3` |
| [meta](meta.md) | 附加自定义元数据 | — |
| [normalizer](normalizer.md) | keyword 字段的归一化处理器 | — |
| [norms](norms.md) | 是否存储字段长度归一化因子 | text: `true` |
| [null_value](null_value.md) | 用指定值替代 null | — |
| [position_increment_gap](position_increment_gap.md) | 数组元素间的位置间隔 | `100` |
| [positive_score_impact](positive_score_impact.md) | rank_feature 对得分的影响方向 | `true` |
| [preserve_position_increments](preserve_position_increments.md) | 完成字段是否保留位置增量 | `true` |
| [preserve_separators](preserve_separators.md) | 完成字段是否保留分隔符 | `true` |
| [properties](properties.md) | 定义对象/嵌套字段的子字段映射 | — |
| [scaling_factor](scaling_factor.md) | scaled_float 字段的缩放因子 | — |
| [search_analyzer](search_analyzer.md) | 查询时使用的分词器 | 同 `analyzer` |
| [search_quote_analyzer](search_quote_analyzer.md) | 带引号短语查询时的分词器 | 同 `search_analyzer` |
| [similarity](similarity.md) | 相关性评分算法 | `BM25` |
| [split_queries_on_whitespace](split_queries_on_whitespace.md) | 查询是否在空白处拆分 | `false` |
| [store](store.md) | 是否独立存储字段值 | `false` |
| [term_vector](term_vector.md) | 是否存储词条向量 | `no` |

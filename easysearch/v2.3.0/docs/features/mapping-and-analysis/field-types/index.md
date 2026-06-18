---
title: "字段类型"
date: 0001-01-01
description: "Easysearch 支持的所有字段类型及其适用场景"
summary: "字段类型 #  Easysearch 提供丰富的字段类型，用于存储和索引不同类型的数据。选择正确的字段类型对查询性能和存储效率至关重要。
 概念指南 → 了解如何设计映射，请阅读：
  映射基础  映射模式与常见坑    字段类型速查 #  文本与关键字 #     类型 说明 典型用途      text 分词索引的全文字段 文章内容、描述、评论    keyword 不分词的精确值 ID、状态码、标签、邮箱    match_only_text 仅用于匹配，节省空间 日志内容（无需评分）    wildcard 高效通配符搜索 日志模式匹配    token_count 存储分词后的词元数量 文本长度统计    选型建议：
 需要全文搜索 → text 需要精确匹配/聚合/排序 → keyword 两者都需要 → 使用 fields 多字段映射  数值类型 #     类型 范围 存储大小 典型用途      byte -128 ~ 127 1 字节 小整数    short -32,768 ~ 32,767 2 字节 短整数    integer -2³¹ ~ 2³¹-1 4 字节 常规整数    long -2⁶³ ~ 2⁶³-1 8 字节 大整数、时间戳    float 32 位浮点 4 字节 常规浮点数    double 64 位浮点 8 字节 高精度浮点    half_float 16 位浮点 2 字节 节省空间的浮点    scaled_float 缩放的长整型 8 字节 金额（scaling_factor=100）    选型建议：选择能容纳数据范围的最小类型，节省存储和内存。"
---


# 字段类型

Easysearch 提供丰富的字段类型，用于存储和索引不同类型的数据。选择正确的字段类型对查询性能和存储效率至关重要。

> **概念指南** → 了解如何设计映射，请阅读：
> - [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
> - [映射模式与常见坑]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

---

## 字段类型速查

### 文本与关键字

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [text](string-field-type/text.md) | 分词索引的全文字段 | 文章内容、描述、评论 |
| [keyword](string-field-type/keyword.md) | 不分词的精确值 | ID、状态码、标签、邮箱 |
| [match_only_text](match_only_text.md) | 仅用于匹配，节省空间 | 日志内容（无需评分） |
| [wildcard](wildcard.md) | 高效通配符搜索 | 日志模式匹配 |
| [token_count](numeric-field.md) | 存储分词后的词元数量 | 文本长度统计 |

**选型建议**：
- 需要全文搜索 → `text`
- 需要精确匹配/聚合/排序 → `keyword`
- 两者都需要 → 使用 `fields` 多字段映射

### 数值类型

| 类型 | 范围 | 存储大小 | 典型用途 |
|------|------|----------|----------|
| [byte](numeric-field.md) | -128 ~ 127 | 1 字节 | 小整数 |
| [short](numeric-field.md) | -32,768 ~ 32,767 | 2 字节 | 短整数 |
| [integer](numeric-field.md) | -2³¹ ~ 2³¹-1 | 4 字节 | 常规整数 |
| [long](numeric-field.md) | -2⁶³ ~ 2⁶³-1 | 8 字节 | 大整数、时间戳 |
| [float](numeric-field.md) | 32 位浮点 | 4 字节 | 常规浮点数 |
| [double](numeric-field.md) | 64 位浮点 | 8 字节 | 高精度浮点 |
| [half_float](numeric-field.md) | 16 位浮点 | 2 字节 | 节省空间的浮点 |
| [scaled_float](numeric-field.md) | 缩放的长整型 | 8 字节 | 金额（scaling_factor=100） |

**选型建议**：选择能容纳数据范围的**最小类型**，节省存储和内存。

### 日期与时间

| 类型 | 精度 | 典型用途 |
|------|------|----------|
| [date](date-field-type/date.md) | 毫秒 | 常规日期时间 |
| [date_nanos](date-field-type/date-nanos.md) | 纳秒 | 高精度时间戳（日志、监控） |

### 布尔与二进制

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [boolean](boolean.md) | true/false | 开关状态、标志位 |
| [binary](binary.md) | Base64 编码二进制 | 文件内容、加密数据 |

### 对象与嵌套

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [object](object-field-type/object.md) | JSON 对象（默认扁平化） | 简单嵌套结构 |
| [nested](object-field-type/nested.md) | 独立索引的对象数组 | 需保持数组元素关联的对象 |
| [flattened](object-field-type/flattened.md) | 扁平化为字符串的对象 | 动态键值对、日志标签 |
| [flattened_text](object-field-type/flattened_text.md) | 扁平化文本对象（Easysearch 扩展） | 动态结构化数据 |
| [join](object-field-type/join.md) | 父子文档关系 | 同索引内的层级关系 |

**选型建议**：
- 简单对象 → `object`
- 对象数组且需要精确匹配单个元素 → `nested`
- 键名动态变化 → `flattened`

### 地理位置

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [geo_point](geo-field-type/geo-point.md) | 经纬度坐标点 | 位置搜索、距离排序 |
| [geo_shape](geo-field-type/geo-shape.md) | 地理形状（多边形等） | 区域搜索、地理围栏 |

### 范围类型

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [integer_range](range-field-type.md) | 整数范围 | 年龄范围、价格区间 |
| [long_range](range-field-type.md) | 长整数范围 | 大数值范围 |
| [float_range](range-field-type.md) | 浮点范围 | 浮点数区间 |
| [double_range](range-field-type.md) | 双精度范围 | 高精度区间 |
| [date_range](range-field-type.md) | 日期范围 | 有效期、预约时段 |
| [ip_range](range-field-type.md) | IP 地址范围 | IP 段匹配 |

### 搜索增强

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [completion](autocomplete-field-type/completion.md) | 补全建议字段 | 搜索框自动补全 |
| [search_as_you_type](autocomplete-field-type/search-as-you-type.md) | 即时搜索 | 边输入边搜索 |
| [rank_feature](rank.md) | 排名特征（单值） | PageRank、热度评分 |
| [rank_features](rank.md) | 排名特征（多值） | 多维度评分 |

### 向量类型

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [knn_vector](knn.md) | k-NN 向量 | 语义搜索、图像相似、向量近邻搜索 |

### 特殊类型

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| [ip](ip.md) | IPv4/IPv6 地址 | IP 地址存储与搜索 |
| [alias](alias.md) | 字段别名 | 兼容旧字段名 |
| [percolator](percolator.md) | 存储查询 | 反向匹配（文档匹配查询） |

### 插件提供的字段类型

| 类型 | 说明 | 典型用途 | 来源 |
|------|------|----------|------|
| `annotated_text` | 带注解的文本 | 文本分析中的内联注释 | mapper-annotated-text 插件 |
| `murmur3` | Murmur3 哈希值 | 内容去重、指纹识别 | mapper-murmur3 插件 |
| `icu_collation_keyword` | ICU 排序关键字 | 多语言排序 | analysis-icu 插件 |

---

## 场景选型指南

| 业务场景 | 推荐类型 | 说明 |
|----------|----------|------|
| 商品标题搜索 | `text` + `keyword` 多字段 | 全文搜索 + 聚合统计 |
| 商品 SKU | `keyword` | 精确匹配 |
| 商品价格 | `scaled_float` (scaling_factor=100) | 金额精确到分 |
| 订单时间 | `date` | 毫秒精度足够 |
| 日志时间 | `date_nanos` | 高精度时间戳 |
| 用户地址 | `geo_point` | 位置搜索 |
| 商品标签 | `keyword` 数组 | 多标签过滤 |
| 嵌套评论 | `nested` | 保持评论完整性 |
| 动态属性 | `flattened` | 键名不固定 |
| 搜索建议 | `completion` | 自动补全 |
| 语义搜索 | `knn_vector` | 向量相似度 |

---

## 数组

Easysearch 没有专门的数组类型。任何字段都可以包含零个或多个值，但数组中所有值必须是相同类型。

```json
PUT testindex/_doc/1
{
  "tags": ["elastic", "search", "engine"]
}
```

---

## 多字段（Multi-fields）

使用 `fields` 参数为同一字段创建多种索引方式：

```json
PUT products
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "keyword": {
            "type": "keyword"
          },
          "pinyin": {
            "type": "text",
            "analyzer": "pinyin"
          }
        }
      }
    }
  }
}
```

- `title` → 全文搜索（中文分词）
- `title.keyword` → 精确匹配、聚合
- `title.pinyin` → 拼音搜索

---

## 空值处理（null_value）

默认情况下，null 值无法被搜索。使用 `null_value` 参数指定替代值：

```json
PUT users
{
  "mappings": {
    "properties": {
      "phone": {
        "type": "keyword",
        "null_value": "N/A"
      }
    }
  }
}
```

索引文档时，`null` 会被替换为 `"N/A"`，从而可以通过 `term` 查询搜索到。

---

## 动态映射

Easysearch 默认开启动态映射，自动为新字段推断类型：

| JSON 值 | 推断类型 |
|---------|----------|
| `true`/`false` | `boolean` |
| 浮点数 | `float` |
| 整数 | `long` |
| 日期格式字符串 | `date` |
| 其他字符串 | `text` + `keyword` 多字段 |
| 对象 | `object` |

**建议**：生产环境使用显式映射，避免动态推断导致类型不一致。

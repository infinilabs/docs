---
title: "词元计数字段类型（Token Count）"
date: 0001-01-01
summary: "词元计数字段类型（Token Count） #  token_count 是一个特殊的整数字段类型，它在索引时自动计算文本经过分析器处理后产生的词元（token）数量，并将该数量作为字段值存储。这使您可以根据文本的词元数量进行过滤、排序和聚合。
适用场景 #   按内容长度过滤：过滤词元数量在指定范围内的文档（如至少 100 个词的文章） 内容质量评估：短文本可能质量较低，可按词元数量排序 分析器效果评估：了解不同分析器对同一文本产生的词元数量差异  创建映射 #  PUT my_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;fields&#34;: { &#34;length&#34;: { &#34;type&#34;: &#34;token_count&#34;, &#34;analyzer&#34;: &#34;standard&#34; } } } } } } 映射参数 #     参数 必填 说明     analyzer ✅ 是 用于分析文本的分析器名称。使用该分析器产生的词元数量作为字段值   enable_position_increments 否 是否计算位置增量。默认 true。设为 false 后，停用词等位置增量不计入总数   doc_values 否 默认 true，启用 doc values 以支持排序和聚合   index 否 默认 true，是否索引该字段   null_value 否 当原始文本为 null 时使用的替代值   store 否 默认 false，是否单独存储字段值    索引文档 #  PUT my_index/_doc/1 { &#34;title&#34;: &#34;快速入门 Easysearch 搜索引擎&#34; } PUT my_index/_doc/2 { &#34;title&#34;: &#34;Easysearch&#34; } PUT my_index/_doc/3 { &#34;title&#34;: &#34;深入理解 Easysearch 的分布式架构设计与高可用方案&#34; } 查询示例 #  过滤词元数量 #  查找标题至少有 3 个词元的文档："
---


# 词元计数字段类型（Token Count）

`token_count` 是一个特殊的整数字段类型，它在索引时自动计算文本经过分析器处理后产生的词元（token）数量，并将该数量作为字段值存储。这使您可以根据文本的词元数量进行过滤、排序和聚合。

## 适用场景

- **按内容长度过滤**：过滤词元数量在指定范围内的文档（如至少 100 个词的文章）
- **内容质量评估**：短文本可能质量较低，可按词元数量排序
- **分析器效果评估**：了解不同分析器对同一文本产生的词元数量差异

## 创建映射

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "length": {
            "type": "token_count",
            "analyzer": "standard"
          }
        }
      }
    }
  }
}
```

### 映射参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `analyzer` | ✅ 是 | 用于分析文本的分析器名称。使用该分析器产生的词元数量作为字段值 |
| `enable_position_increments` | 否 | 是否计算位置增量。默认 `true`。设为 `false` 后，停用词等位置增量不计入总数 |
| `doc_values` | 否 | 默认 `true`，启用 doc values 以支持排序和聚合 |
| `index` | 否 | 默认 `true`，是否索引该字段 |
| `null_value` | 否 | 当原始文本为 null 时使用的替代值 |
| `store` | 否 | 默认 `false`，是否单独存储字段值 |

## 索引文档

```json
PUT my_index/_doc/1
{
  "title": "快速入门 Easysearch 搜索引擎"
}

PUT my_index/_doc/2
{
  "title": "Easysearch"
}

PUT my_index/_doc/3
{
  "title": "深入理解 Easysearch 的分布式架构设计与高可用方案"
}
```

## 查询示例

### 过滤词元数量

查找标题至少有 3 个词元的文档：

```json
GET my_index/_search
{
  "query": {
    "range": {
      "title.length": {
        "gte": 3
      }
    }
  }
}
```

### 按词元数量排序

```json
GET my_index/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "title.length": "desc" }
  ]
}
```

### 聚合统计

```json
GET my_index/_search
{
  "size": 0,
  "aggs": {
    "title_length_stats": {
      "stats": {
        "field": "title.length"
      }
    },
    "title_length_distribution": {
      "histogram": {
        "field": "title.length",
        "interval": 5
      }
    }
  }
}
```

## enable_position_increments 参数

当设置 `enable_position_increments: false` 时，分析器产生的位置增量（如停用词被移除留下的空位）不会计入词元总数。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "fields": {
          "word_count": {
            "type": "token_count",
            "analyzer": "english",
            "enable_position_increments": false
          }
        }
      }
    }
  }
}
```

对于文本 `"The quick brown fox"`，使用 `english` 分析器：
- `enable_position_increments: true`（默认）：计数为 4（包含停用词 "the" 的位置）
- `enable_position_increments: false`：计数为 3（不包含已移除的 "the"）

## 使用不同分析器

`token_count` 字段的 `analyzer` 可以与父字段使用不同的分析器：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "cn_words": {
            "type": "token_count",
            "analyzer": "ik_smart"
          },
          "std_tokens": {
            "type": "token_count",
            "analyzer": "standard"
          }
        }
      }
    }
  }
}
```

这样可以同时统计不同分析器的词元数量，用于分析或对比。


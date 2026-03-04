---
title: "文档大小元数据字段（_size）"
date: 0001-01-01
summary: "文档大小元数据字段（_size） #  _size 元数据字段记录每个文档的 _source 字段的原始未压缩大小（字节数）。启用后，可以按文档大小进行过滤、排序和聚合。
前置条件 #  _size 字段由 mapper-size 插件提供，需要确认插件已安装：
bin/easysearch-plugin list # 应包含 mapper-size 启用 _size 字段 #  _size 字段默认未启用，需要在索引映射中显式开启：
PUT my_index { &#34;mappings&#34;: { &#34;_size&#34;: { &#34;enabled&#34;: true } } } 使用示例 #  索引文档 #  启用 _size 后，索引文档时会自动计算并存储 _source 的大小：
PUT my_index/_doc/1 { &#34;title&#34;: &#34;示例文档&#34;, &#34;content&#34;: &#34;这是一个示例文档，用于演示 _size 字段。&#34; } 查询文档大小 #  GET my_index/_search { &#34;query&#34;: { &#34;match_all&#34;: {} }, &#34;fields&#34;: [&#34;_size&#34;], &#34;_source&#34;: false } 按大小过滤 #  查找大于 1KB 的文档："
---


# 文档大小元数据字段（_size）

`_size` 元数据字段记录每个文档的 `_source` 字段的原始未压缩大小（字节数）。启用后，可以按文档大小进行过滤、排序和聚合。

## 前置条件

`_size` 字段由 `mapper-size` 插件提供，需要确认插件已安装：

```bash
bin/easysearch-plugin list
# 应包含 mapper-size
```

## 启用 _size 字段

`_size` 字段默认未启用，需要在索引映射中显式开启：

```json
PUT my_index
{
  "mappings": {
    "_size": {
      "enabled": true
    }
  }
}
```

## 使用示例

### 索引文档

启用 `_size` 后，索引文档时会自动计算并存储 `_source` 的大小：

```json
PUT my_index/_doc/1
{
  "title": "示例文档",
  "content": "这是一个示例文档，用于演示 _size 字段。"
}
```

### 查询文档大小

```json
GET my_index/_search
{
  "query": { "match_all": {} },
  "fields": ["_size"],
  "_source": false
}
```

### 按大小过滤

查找大于 1KB 的文档：

```json
GET my_index/_search
{
  "query": {
    "range": {
      "_size": {
        "gt": 1024
      }
    }
  }
}
```

### 按大小排序

```json
GET my_index/_search
{
  "sort": [
    { "_size": "desc" }
  ]
}
```

### 统计文档大小分布

```json
GET my_index/_search
{
  "size": 0,
  "aggs": {
    "size_stats": {
      "stats": {
        "field": "_size"
      }
    },
    "size_distribution": {
      "histogram": {
        "field": "_size",
        "interval": 1024
      }
    }
  }
}
```

## 适用场景

- **监控文档大小**：识别异常大的文档，防止单个文档过大影响性能
- **容量规划**：分析文档大小分布，预估存储需求
- **数据质量检查**：过滤过小（可能为空或不完整）或过大（可能有问题）的文档
- **成本优化**：按文档大小排序，找出占用存储最多的文档

## 注意事项

- `_size` 存储的是 `_source` 的原始字节大小，不包括 Lucene 索引结构的大小
- 启用 `_size` 会略微增加索引体积（每个文档多存储一个整数字段）
- 只能在创建索引时或通过更新映射启用，不会对已有文档回填大小值


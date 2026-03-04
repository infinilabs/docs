---
title: "哈希字段类型（Murmur3）"
date: 0001-01-01
summary: "哈希字段类型（Murmur3） #  murmur3 字段类型在索引时计算字段值的 MurmurHash3 128 位哈希值，并将哈希值存储为 doc values。这在 cardinality 聚合中可以提供更高的精度，因为直接使用预计算的哈希值而非运行时计算。
前置条件 #  murmur3 字段类型由 mapper-murmur3 插件提供，需要确认插件已安装：
bin/easysearch-plugin list # 应包含 mapper-murmur3 创建映射 #  PUT my_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;user_id&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;fields&#34;: { &#34;hash&#34;: { &#34;type&#34;: &#34;murmur3&#34; } } } } } } 使用示例 #  Cardinality 聚合优化 #  使用 murmur3 子字段进行基数统计，可以获得更高的精度：
GET my_index/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;unique_users&#34;: { &#34;cardinality&#34;: { &#34;field&#34;: &#34;user_id."
---


# 哈希字段类型（Murmur3）

`murmur3` 字段类型在索引时计算字段值的 MurmurHash3 128 位哈希值，并将哈希值存储为 doc values。这在 `cardinality` 聚合中可以提供更高的精度，因为直接使用预计算的哈希值而非运行时计算。

## 前置条件

`murmur3` 字段类型由 `mapper-murmur3` 插件提供，需要确认插件已安装：

```bash
bin/easysearch-plugin list
# 应包含 mapper-murmur3
```

## 创建映射

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword",
        "fields": {
          "hash": {
            "type": "murmur3"
          }
        }
      }
    }
  }
}
```

## 使用示例

### Cardinality 聚合优化

使用 `murmur3` 子字段进行基数统计，可以获得更高的精度：

```json
GET my_index/_search
{
  "size": 0,
  "aggs": {
    "unique_users": {
      "cardinality": {
        "field": "user_id.hash"
      }
    }
  }
}
```

## 限制

- `murmur3` 字段不支持搜索查询（不可检索）
- 只能用于聚合和排序
- 字段值在索引时计算并存储，会增加少量索引体积
- 主要用途是优化 `cardinality` 聚合的精度


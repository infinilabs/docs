---
title: "扁平化文本字段类型（Flattened Text）"
date: 0001-01-01
summary: "Flattened Text 字段类型 #  flattened_text 类型是一种特殊的数据结构，适用于存储和查询嵌套层次的数据，同时保留类似于 text 类型的灵活搜索特性，例如分词和全文匹配。它在处理结构化或者半结构化数据时非常有用，例如 JSON 对象或动态键值对映射。
相关指南（先读这些） #    映射基础  映射模式  全文搜索  定义映射 #  { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;flattened_text&#34; } } } 特性 #    扁平化存储
 将嵌套的 JSON 对象转换为扁平结构 保留完整的路径信息 支持点号访问内部字段    文本分析
 支持标准分词器 支持短语查询 支持全文搜索功能    内部索引结构 每个 flattened_text 字段在 lucene 层面会创建多个子字段:
 {field} - 存储所有键 {field}._value - 存储所有值 {field}."
---


# Flattened Text 字段类型

`flattened_text` 类型是一种特殊的数据结构，适用于存储和查询嵌套层次的数据，同时保留类似于 `text` 类型的灵活搜索特性，例如分词和全文匹配。它在处理结构化或者半结构化数据时非常有用，例如 JSON 对象或动态键值对映射。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

## 定义映射

```json
{
  "properties": {
    "my_field": {
      "type": "flattened_text"  
    }
  }
}
```

## 特性

1. 扁平化存储
    - 将嵌套的 JSON 对象转换为扁平结构
    - 保留完整的路径信息
    - 支持点号访问内部字段

2. 文本分析
    - 支持标准分词器
    - 支持短语查询
    - 支持全文搜索功能

3. 内部索引结构
   每个 `flattened_text` 字段在 lucene 层面会创建多个子字段:
    - `{field}` - 存储所有键
    - `{field}._value` - 存储所有值
    - `{field}._valueAndPath` - 存储 "path=value" 格式

## 索引示例

```auto
PUT my_index/_doc/1
{
  "my_field": {
    "key1": {
      "subkey1": {
        "subkey2": "quick brown fox"
      }
    }
  }
}
```

## 查询示例
### 精确路径匹配
```auto
// Match Query - 指定完整路径
GET my_index/_search
{
  "query": {
    "match": {
      "my_field.key1.subkey1.subkey2": "quick brown fox"
    }
  }
}

// Match Phrase Query - 指定完整路径的短语匹配
GET my_index/_search
{
  "query": {
    "match_phrase": {
      "my_field.key1.subkey1.subkey2": "quick brown fox"
    }
  }
}
```

### 通过根字段字段查询
```auto
// Match Query - 匹配所有值
GET my_index/_search
{
  "query": {
    "match": {
      "my_field": "quick brown fox"
    }
  }
}

// Match Phrase Query - 短语匹配所有值
GET my_index/_search
{
  "query": {
    "match_phrase": {
      "my_field": "quick brown fox" 
    }
  }
}
```


## 使用场景

1. 需要对 JSON 对象进行全文检索
2. 需要保持对象结构但又想避免字段爆炸
3. 对象结构不固定或未知

## 限制

1. 不支持聚合操作
2. 不支持排序
3. 不支持通配符查询

## 对比

与普通 `text` 和 `flattened` 类型相比:

- vs text:支持嵌套对象访问
- vs flattened:支持全文检索功能

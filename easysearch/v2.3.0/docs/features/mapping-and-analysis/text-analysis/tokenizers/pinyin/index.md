---
title: "拼音分词器（Pinyin）"
date: 0001-01-01
summary: "Pinyin 分词器 #  pinyin 分词器来自 analysis-pinyin 插件，将中文文本直接转换为拼音词元。与 pinyin 词元过滤器不同，分词器在分词阶段就完成拼音转换。
前提条件 #  bin/easysearch-plugin install analysis-pinyin 分词器 vs 过滤器 #     组件 说明 适用场景     pinyin 分词器 直接将整段文本转拼音并分词 纯拼音索引   pinyin 词元过滤器 在已有分词结果上转拼音 需要保留原始中文词元    使用示例 #  基本用法 #  PUT my-pinyin-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;pinyin_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;pinyin&#34;, &#34;filter&#34;: [&#34;lowercase&#34;] } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;pinyin_analyzer&#34; } } } } 测试分词 #  GET /_analyze { &#34;tokenizer&#34;: &#34;pinyin&#34;, &#34;text&#34;: &#34;中华人民共和国&#34; } 参数 #  分词器支持与 pinyin 词元过滤器相同的参数："
---


# Pinyin 分词器

`pinyin` 分词器来自 analysis-pinyin 插件，将中文文本直接转换为拼音词元。与 `pinyin` 词元过滤器不同，分词器在分词阶段就完成拼音转换。

## 前提条件

```bash
bin/easysearch-plugin install analysis-pinyin
```

## 分词器 vs 过滤器

| 组件 | 说明 | 适用场景 |
|------|------|---------|
| **`pinyin` 分词器** | 直接将整段文本转拼音并分词 | 纯拼音索引 |
| `pinyin` 词元过滤器 | 在已有分词结果上转拼音 | 需要保留原始中文词元 |

## 使用示例

### 基本用法

```json
PUT my-pinyin-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "pinyin_analyzer": {
          "type": "custom",
          "tokenizer": "pinyin",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "pinyin_analyzer"
      }
    }
  }
}
```

### 测试分词

```json
GET /_analyze
{
  "tokenizer": "pinyin",
  "text": "中华人民共和国"
}
```

## 参数

分词器支持与 `pinyin` 词元过滤器相同的参数：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `keep_first_letter` | `true` | 保留拼音首字母 |
| `keep_full_pinyin` | `true` | 保留全拼 |
| `keep_joined_full_pinyin` | `false` | 连接全拼输出 |
| `keep_original` | `false` | 保留原始中文 |
| `keep_none_chinese` | `true` | 保留非中文字符 |
| `limit_first_letter_length` | `16` | 首字母最大长度 |
| `lowercase` | `true` | 拼音小写 |
| `remove_duplicated_term` | `false` | 移除重复词项 |

## 相关链接

- [拼音过滤器]({{< relref "../token-filters/pinyin-filter" >}}) — 作为词元过滤器使用
- [拼音分析器]({{< relref "../analyzers/pinyin-analyzer" >}}) — 预配置的拼音分析器
- [拼音首字母分词器]({{< relref "./pinyin-first-letter" >}}) — 仅首字母模式
- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


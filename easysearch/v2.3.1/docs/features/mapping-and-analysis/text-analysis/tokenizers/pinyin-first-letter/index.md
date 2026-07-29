---
title: "拼音首字母分词器（Pinyin First Letter）"
date: 0001-01-01
summary: "Pinyin First Letter 分词器 #  pinyin_first_letter 分词器将中文字符转换为拼音首字母缩写，例如 &ldquo;中华人民共和国&rdquo; → &ldquo;zhrmghg&rdquo;。适用于拼音首字母搜索场景。
需要安装 analysis-pinyin 插件。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;pinyin_first_letter&#34; } } } } } 相关指南 #    Pinyin 分词器  Pinyin 分析器  文本分析基础  "
---


# Pinyin First Letter 分词器

`pinyin_first_letter` 分词器将中文字符转换为拼音首字母缩写，例如 "中华人民共和国" → "zhrmghg"。适用于拼音首字母搜索场景。

需要安装 `analysis-pinyin` 插件。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "pinyin_first_letter"
        }
      }
    }
  }
}
```

## 相关指南

- [Pinyin 分词器]({{< relref "./pinyin" >}})
- [Pinyin 分析器]({{< relref "../analyzers/pinyin-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


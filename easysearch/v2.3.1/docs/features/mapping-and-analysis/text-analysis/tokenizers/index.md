---
title: "分词器（Tokenizers）"
date: 0001-01-01
description: "分词器将文本拆分为词元，是文本分析链的核心组件"
summary: "分词器（Tokenizers） #  分词器是分析链的核心组件，负责将字符流拆分为词元（tokens）。每个分析器有且只有一个分词器。
 概念指南 → 深入理解分词原理，请阅读 词汇识别
  分词器工作原理 #  输入文本: &#34;Actions speak louder than words.&#34; ↓ [分词器处理] ↓ 词元流: [Actions, speak, louder, than, words] + 位置信息 + 偏移量 + 类型标签 分词器不仅拆分文本，还会维护词元的元数据：
   元数据 用途 示例     位置（position） 短语查询、邻近查询 speak 位于位置 2   偏移量（offset） 搜索结果高亮 speak 起始于字符 8   类型（type） 分类过滤 &lt;ALPHANUM&gt;, &lt;NUM&gt;     分词器速查表 #  单词分词器（Word-Oriented） #  适用于全文搜索场景，将文本拆分为独立单词。"
---


# 分词器（Tokenizers）

分词器是分析链的核心组件，负责将字符流拆分为词元（tokens）。每个分析器有且只有一个分词器。

> **概念指南** → 深入理解分词原理，请阅读 [词汇识别]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

---

## 分词器工作原理

```
输入文本: "Actions speak louder than words."
            ↓
       [分词器处理]
            ↓
词元流: [Actions, speak, louder, than, words]
        + 位置信息 + 偏移量 + 类型标签
```

分词器不仅拆分文本，还会维护词元的元数据：

| 元数据 | 用途 | 示例 |
|--------|------|------|
| **位置（position）** | 短语查询、邻近查询 | `speak` 位于位置 2 |
| **偏移量（offset）** | 搜索结果高亮 | `speak` 起始于字符 8 |
| **类型（type）** | 分类过滤 | `<ALPHANUM>`, `<NUM>` |

---

## 分词器速查表

### 单词分词器（Word-Oriented）

适用于全文搜索场景，将文本拆分为独立单词。

| 分词器 | 说明 | 推荐场景 |
|--------|------|----------|
| [standard](standard/) | Unicode 标准分词，移除标点 | **通用首选**，支持大多数语言 |
| [letter](letter/) | 非字母处拆分 | 纯字母文本 |
| [lowercase](lowercase/) | letter + 小写转换 | 大小写不敏感搜索 |
| [whitespace](whitespace/) | 仅按空白拆分 | 保留标点符号 |
| [uax_url_email](uax-url-email/) | 保留 URL 和邮箱完整性 | 含链接或邮箱的文本 |
| [classic](classic/) | 英文语法规则，保留缩写 | 英文文档、邮件 |

### 部分匹配分词器（Partial Word）

生成词元片段，支持前缀匹配、模糊搜索等场景。

| 分词器 | 说明 | 典型输出 |
|--------|------|----------|
| [ngram](n-gram/) | 生成 n-gram 序列 | `quick` → `[q, qu, ui, ic, ck, ...]` |
| [edge_ngram](edge-n-gram/) | 从开头生成 n-gram | `quick` → `[q, qu, qui, quic, quick]` |

**选型建议**：
- **即时搜索（search-as-you-type）** → `edge_ngram`
- **模糊匹配、拼写纠错** → `ngram`

### 结构化文本分词器（Structured Text）

处理标识符、路径、模式等结构化内容。

| 分词器 | 说明 | 推荐场景 |
|--------|------|----------|
| [keyword](keyword/) | 不分词，整体作为单个词元 | 精确匹配（ID、SKU） |
| [pattern](pattern/) | 正则表达式拆分 | 自定义分隔规则 |
| [simple_pattern_split](simple-pattern-split/) | Lucene 正则，性能更优 | 高性能正则分词 |
| [char_group](character-group/) | 按字符集拆分 | 简单字符分隔 |
| [path_hierarchy](path-hierarchy/) | 路径层级拆分 | 文件路径、分类树 |

---

## 分词器选型指南

| 应用场景 | 推荐分词器 | 说明 |
|----------|------------|------|
| 通用全文搜索 | `standard` | 默认选择，开箱即用 |
| 中文搜索 | 配合中文分析器 | 见 [中文分析器](../analyzers/) |
| 即时搜索提示 | `edge_ngram` | 支持前缀匹配 |
| 精确 ID 匹配 | `keyword` | 不分词 |
| 文件路径搜索 | `path_hierarchy` | 层级展开 |
| 保留 URL 完整 | `uax_url_email` | 不拆分链接 |
| 日志分析 | `pattern` / `char_group` | 按自定义分隔符 |

---

## 自定义分词器示例

### edge_ngram 实现即时搜索

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "autocomplete_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": ["letter", "digit"]
        }
      },
      "analyzer": {
        "autocomplete": {
          "type": "custom",
          "tokenizer": "autocomplete_tokenizer",
          "filter": ["lowercase"]
        }
      }
    }
  }
}
```

### path_hierarchy 处理文件路径

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "path_tokenizer": {
          "type": "path_hierarchy",
          "delimiter": "/",
          "reverse": false
        }
      }
    }
  }
}
```

输入 `/usr/local/bin` → 输出 `[/usr, /usr/local, /usr/local/bin]`

---

## 测试分词器

使用 `_analyze` API 测试分词效果：

```json
POST /_analyze
{
  "tokenizer": "standard",
  "text": "Hello, World! This is Easysearch."
}
```

测试自定义分词器：

```json
POST /my-index/_analyze
{
  "tokenizer": "autocomplete_tokenizer",
  "text": "easysearch"
}
```

---

## 分词器完整列表

### 单词分词器

| 分词器 | 描述 |
|--------|------|
| [standard]({{< relref "./standard" >}}) | 在单词边界处拆分，移除大部分标点符号 |
| [letter]({{< relref "./letter" >}}) | 在非字母字符处拆分 |
| [lowercase]({{< relref "./lowercase" >}}) | letter + 小写转换 |
| [whitespace]({{< relref "./whitespace" >}}) | 仅在空白字符处拆分 |
| [uax_url_email]({{< relref "./uax-url-email" >}}) | standard 变体，保留 URL 和邮箱完整 |
| [classic]({{< relref "./classic" >}}) | 基于语法的英文分词器 |

### 部分匹配分词器

| 分词器 | 描述 |
|--------|------|
| [ngram]({{< relref "./n-gram" >}}) | 生成指定长度范围的 n-gram |
| [edge_ngram]({{< relref "./edge-n-gram" >}}) | 从词元开头生成 n-gram |
| [simple_pattern]({{< relref "./simple-pattern" >}}) | Lucene 正则匹配（不拆分） |

### 结构化文本分词器

| 分词器 | 描述 |
|--------|------|
| [keyword]({{< relref "./keyword" >}}) | 不分词，整体输出 |
| [pattern]({{< relref "./pattern" >}}) | Java 正则表达式分词 |
| [simple_pattern_split]({{< relref "./simple-pattern-split" >}}) | Lucene 正则拆分，性能更优 |
| [char_group]({{< relref "./character-group" >}}) | 按指定字符集拆分 |
| [path_hierarchy]({{< relref "./path-hierarchy" >}}) | 按路径分隔符层级拆分 |

### 国际化分词器

| 分词器 | 描述 | 插件 |
|--------|------|------|
| [thai]({{< relref "./thai" >}}) | 泰语分词（Java BreakIterator） | 内置 |
| [icu_tokenizer]({{< relref "./icu-tokenizer" >}}) | ICU 多语言分词 | analysis-icu |

### 中文分词器

| 分词器 | 描述 | 插件 |
|--------|------|------|
| [ik_smart]({{< relref "./ik-smart" >}}) | IK 智能分词（粗粒度） | analysis-ik |
| [ik_max_word]({{< relref "./ik-max-word" >}}) | IK 最大化分词（细粒度） | analysis-ik |
| [jieba_search]({{< relref "./jieba-search" >}}) | Jieba 搜索模式 | analysis-jieba |
| [jieba_index]({{< relref "./jieba-index" >}}) | Jieba 索引模式 | analysis-jieba |
| [pinyin]({{< relref "./pinyin" >}}) | 拼音分词 | analysis-pinyin |
| [pinyin_first_letter]({{< relref "./pinyin-first-letter" >}}) | 拼音首字母分词 | analysis-pinyin |
| [hanlp]({{< relref "./hanlp" >}}) | HanLP 通用分词 | analysis-hanlp |
| [hanlp_standard]({{< relref "./hanlp-standard" >}}) | HanLP 标准分词 | analysis-hanlp |
| [hanlp_index]({{< relref "./hanlp-index" >}}) | HanLP 索引分词 | analysis-hanlp |
| [hanlp_nlp]({{< relref "./hanlp-nlp" >}}) | HanLP NLP 分词 | analysis-hanlp |
| [hanlp_crf]({{< relref "./hanlp-crf" >}}) | HanLP CRF 分词 | analysis-hanlp |
| [hanlp_n_short]({{< relref "./hanlp-n-short" >}}) | HanLP N-最短路径 | analysis-hanlp |
| [hanlp_dijkstra]({{< relref "./hanlp-dijkstra" >}}) | HanLP 最短路径 | analysis-hanlp |
| [hanlp_speed]({{< relref "./hanlp-speed" >}}) | HanLP 极速分词 | analysis-hanlp |
| [stconvert]({{< relref "./stconvert" >}}) | 简繁转换分词 | analysis-stconvert |

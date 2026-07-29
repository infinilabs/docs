---
title: "文本分析"
date: 0001-01-01
description: "文本分析组件、语言分析器选择与配置、分析器、分词器、过滤器参考。"
summary: "📌 基础概念：请先阅读 Mapping 与文本分析 了解文本分析的三步处理（字符过滤 → 分词 → Token 过滤）。
 文本分析 #  文本分析是将原始文本转换为可搜索词项的过程。Easysearch 提供了丰富的分析组件，可以自由组合构建分析链。
语言分析器概览 #  Easysearch 为多种流行语言提供了开箱即用的语言分析器。这些分析器针对各自语言的特点进行了优化：
   语言 分析器 特点 支持清单     英文 english 移除所有格、词干提取 ✅   法文 french 移除省略、变音符号 ✅   德文 german 字符规范化（ä→a） ✅   中文 ik、pinyin（需插件） 分词、拼音 ✅ 扩展   日文 需安装插件 汉字、假名处理 扩展   其他 阿拉伯、西班牙、俄语等 各语言优化 ✅    使用语言分析器 #  方法 1：在映射中指定"
---


> 📌 **基础概念**：请先阅读 [Mapping 与文本分析]({{< relref "/docs/fundamentals/mapping-analysis-intro.md" >}}) 了解文本分析的三步处理（字符过滤 → 分词 → Token 过滤）。

# 文本分析

文本分析是将原始文本转换为可搜索词项的过程。Easysearch 提供了丰富的分析组件，可以自由组合构建分析链。

## 语言分析器概览

Easysearch 为多种流行语言提供了开箱即用的语言分析器。这些分析器针对各自语言的特点进行了优化：

| 语言 | 分析器 | 特点 | 支持清单 |
|------|--------|------|--------|
| **英文** | `english` | 移除所有格、词干提取 | ✅ |
| **法文** | `french` | 移除省略、变音符号 | ✅ |
| **德文** | `german` | 字符规范化（ä→a） | ✅ |
| **中文** | `ik`、`pinyin`（需插件） | 分词、拼音 | ✅ 扩展 |
| **日文** | 需安装插件 | 汉字、假名处理 | 扩展 |
| **其他** | 阿拉伯、西班牙、俄语等 | 各语言优化 | ✅ |

### 使用语言分析器

**方法 1：在映射中指定**

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english"
      },
      "content": {
        "type": "text",
        "analyzer": "french"
      },
      "description": {
        "type": "text",
        "analyzer": "standard"
      }
    }
  }
}
```

**方法 2：在搜索时指定**

```json
GET /my_index/_search
{
  "query": {
    "match": {
      "title": {
        "query": "quick brown fox",
        "analyzer": "english"
      }
    }
  }
}
```

> ⚠️ **重要**：索引和搜索必须使用相同的分析器，才能正确匹配词条。

## 常用语言分析器详解

### 英语分析器 (english)

处理特点：
- 移除停用词：The, a, an, is, at 等
- 词干提取：foxes → fox, jumped → jump, quickly → quick
- 移除所有格：John's → john

```json
"The quick brown foxes jumped" → ["quick", "brown", "fox", "jump"]
```

### 法语分析器 (french)

处理特点：
- 移除法文停用词：le, la, les, de, du 等
- 移除元音省略：`l'` 和 `qu'`
- 移除变音符号：café → cafe

### 德语分析器 (german)

处理特点：
- 德语停用词处理
- 字符规范化：ä/ö/ü → a/o/u，ß → ss
- 词干提取

### 标准分析器 (standard)

**默认分析器**，语言通用：
- 以 Unicode 边界为标准分词（空格、标点）
- 转小写
- 不删除停用词
- 无语言特定的词干提取

> 当你无法确定文本所用语言，或文本包含多种语言时，用标准分析器。

## 分析链结构

一个完整的分析器（Analyzer）由三部分组成：

```
原始文本
    ↓
┌─────────────────────┐
│   字符过滤器        │  ← 预处理：移除 HTML、字符替换等
│ (Character Filters) │
└─────────────────────┘
    ↓
┌─────────────────────┐
│     分词器          │  ← 拆分：将文本切分为词项
│   (Tokenizer)       │
└─────────────────────┘
    ↓
┌─────────────────────┐
│   词元过滤器        │  ← 后处理：小写、词干、同义词等
│  (Token Filters)    │
└─────────────────────┘
    ↓
可搜索的词项 (Terms)
```

## 组件分类

| 组件类型 | 作用 | 数量限制 | 常用示例 |
|---------|------|---------|---------|
| [分析器]({{< relref "./analyzers" >}}) | 预定义的完整分析链 | 1 个 | standard、ik、pinyin |
| [分词器]({{< relref "./tokenizers" >}}) | 将文本拆分为词项 | 必须 1 个 | standard、whitespace、pattern |
| [字符过滤器]({{< relref "./character-filters" >}}) | 预处理原始文本 | 0~多个 | html_strip、mapping |
| [词元过滤器]({{< relref "./token-filters" >}}) | 处理分词后的词项 | 0~多个 | lowercase、stemmer、synonym |
| [规范化器]({{< relref "./normalizers" >}}) | keyword 字段的字符级归一化 | 0~1 个 | lowercase、自定义 |

## 快速参考

### 内置分析器

| 分析器 | 说明 | 适用场景 |
|-------|------|---------|
| [标准分析器（Standard）]({{< relref "./analyzers/standard-analyzer" >}}) | 默认分析器，按单词边界分词 | 通用英文文本 |
| [简单分析器（Simple）]({{< relref "./analyzers/simple-analyzer" >}}) | 按非字母字符分词 | 简单文本 |
| [空白分析器（Whitespace）]({{< relref "./analyzers/whitespace-analyzer" >}}) | 仅按空白字符分词 | 保留标点的场景 |
| [IK 中文分析器]({{< relref "./analyzers/ik-analyzer" >}}) | 中文智能分词 | 中文全文搜索 |
| [拼音分析器（Pinyin）]({{< relref "./analyzers/pinyin-analyzer" >}}) | 中文转拼音 | 拼音搜索 |
| [语言分析器（Language）]({{< relref "./analyzers/language-analyzer" >}}) | 特定语言优化 | 多语言支持 |

### 常用分词器

| 分词器 | 说明 |
|-------|------|
| [标准分词器（Standard）]({{< relref "./tokenizers/standard" >}}) | Unicode 文本分割，适合大多数语言 |
| [空白分词器（Whitespace）]({{< relref "./tokenizers/whitespace" >}}) | 仅按空白字符分割 |
| [N-gram 分词器]({{< relref "./tokenizers/n-gram" >}}) | 生成字符级 n-gram |
| [Edge N-gram 分词器]({{< relref "./tokenizers/edge-n-gram" >}}) | 从词首生成 n-gram，适合自动补全 |
| [正则分词器（Pattern）]({{< relref "./tokenizers/pattern" >}}) | 按正则表达式分割 |

### 常用词元过滤器

| 过滤器 | 说明 |
|-------|------|
| [小写过滤器（Lowercase）]({{< relref "./token-filters/lowercase" >}}) | 转换为小写 |
| [词干过滤器（Stemmer）]({{< relref "./token-filters/stemmer" >}}) | 提取词干 |
| [停用词过滤器（Stop）]({{< relref "./token-filters/stop" >}}) | 移除停用词 |
| [同义词过滤器（Synonym）]({{< relref "./token-filters/synonym" >}}) | 同义词扩展 |
| [ASCII 折叠（ASCII Folding）]({{< relref "./token-filters/ascii-folding" >}}) | 移除变音符号 |
| [CJK 二元组（CJK Bigram）]({{< relref "./token-filters/cjk-bigram" >}}) | CJK 字符双字组合 |

### 规范化器（Normalizers）

规范化器（Normalizer）类似分析器，但**只输出单个词元**，专用于 `keyword` 字段的字符级归一化。

| 规范化器 | 说明 |
|----------|------|
| [lowercase]({{< relref "./normalizers/lowercase" >}}) | 内置规范化器，将整个值转为小写 |
| [自定义规范化器]({{< relref "./normalizers/custom" >}}) | 组合字符过滤器和词元过滤器构建自定义归一化逻辑 |

> 详见 [规范化器参考]({{< relref "./normalizers" >}})

## 选择分析器的建议

| 场景 | 推荐分析器 | 说明 |
|------|-----------|------|
| **英文内容** | `english` | 充分利用词干提取提升召回 |
| **多语言混合** | `standard` | 避免误判，通用分析 |
| **中文内容** | `ik`（需插件） | 专业分词效果更好 |
| **商品名称、品牌词** | `standard` 或 `keyword` | 避免过度分析 |
| **实验性选择** | 使用 `_analyze` API 对比 | 基于实际文本效果选择 |

## 测试分析器

使用 `_analyze` API 测试不同语言分析器的效果：

```json
GET /_analyze
{
  "analyzer": "english",
  "text": "The quick brown foxes"
}
```

响应示例：

```json
{
  "tokens": [
    { "token": "quick",   "position": 1 },
    { "token": "brown",   "position": 2 },
    { "token": "fox",     "position": 3 }
  ]
}
```

对比不同分析器：

```json
GET /_analyze
{
  "analyzer": "english",
  "text": "The foxes jumped quickly"
}
```

结果：`["fox", "jump", "quick"]`（停用词移除 + 词干提取）

```json
GET /_analyze
{
  "analyzer": "standard",
  "text": "The foxes jumped quickly"
}
```

结果：`["the", "foxes", "jumped", "quickly"]`（仅分词，不做语言处理）

```json
```

## 自定义分析器

如果内置分析器不满足需求，可自定义：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": ["html_strip"],
          "filter": [
            "lowercase",
            "stop",
            "snowball"
          ]
        }
      }
    }
  }
}
```

## 相关指南

深入学习文本分析主题：

- [词汇识别]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words" >}}) - 分词器选择
- [词元归一化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization" >}}) - 归一化处理
- [词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming" >}}) - 词干提取策略
- [停用词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stopwords" >}}) - 停用词配置
- [同义词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-synonyms" >}}) - 同义词配置
- [模糊匹配]({{< relref "/docs/features/mapping-and-analysis/text-analysis-fuzzy" >}}) - 模糊匹配

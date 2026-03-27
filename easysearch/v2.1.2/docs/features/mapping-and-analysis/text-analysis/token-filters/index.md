---
title: "词元过滤器（Token Filters）"
date: 0001-01-01
description: "词元过滤器用于修改、添加或删除词元"
summary: "词元过滤器（Token Filters） #  词元过滤器是分析链的最后阶段，对分词器产生的词元进行增强处理。一个分析器可以配置多个词元过滤器，按顺序执行。
 概念指南 → 深入理解各类过滤器原理：
  词元归一化 — 大小写、音标折叠  词干提取 — 词形还原  停用词 — 过滤无意义词  同义词 — 扩展查询召回    词元过滤器在分析链中的位置 #  原始文本 ──→ 字符过滤器 ──→ 分词器 ──→ [词元过滤器链] ──→ 索引 ↑ 增强处理阶段 lowercase → stemmer → stop  按功能分类速查 #  大小写与归一化 #     过滤器 说明 典型用途      lowercase 转小写 最常用，大小写不敏感搜索    uppercase 转大写 特殊规范化需求    asciifolding 音标→ASCII café → cafe    cjk_width CJK 全角/半角转换 中日韩文本标准化    词干提取（Stemming） #  将单词还原为词根，提升召回率。"
---


# 词元过滤器（Token Filters）

词元过滤器是分析链的**最后阶段**，对分词器产生的词元进行增强处理。一个分析器可以配置多个词元过滤器，按顺序执行。

> **概念指南** → 深入理解各类过滤器原理：
> - [词元归一化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}}) — 大小写、音标折叠
> - [词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}}) — 词形还原
> - [停用词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stopwords.md" >}}) — 过滤无意义词
> - [同义词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-synonyms.md" >}}) — 扩展查询召回

---

## 词元过滤器在分析链中的位置

```
原始文本 ──→ 字符过滤器 ──→ 分词器 ──→ [词元过滤器链] ──→ 索引
                                        ↑
                                    增强处理阶段
                              lowercase → stemmer → stop
```

---

## 按功能分类速查

### 大小写与归一化

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [lowercase](lowercase/) | 转小写 | **最常用**，大小写不敏感搜索 |
| [uppercase](uppercase/) | 转大写 | 特殊规范化需求 |
| [asciifolding](ascii-folding/) | 音标→ASCII | `café` → `cafe` |
| [cjk_width](cjk-width/) | CJK 全角/半角转换 | 中日韩文本标准化 |

### 词干提取（Stemming）

将单词还原为词根，提升召回率。

| 过滤器 | 说明 | 适用语言 |
|--------|------|----------|
| [stemmer](stemmer/) | 通用词干提取 | 多语言（见配置） |
| [snowball](snowball/) | Snowball 算法 | 欧洲语言 |
| [porter_stem](porter-stem/) | Porter 算法 | 英语 |
| [kstem](kstem/) | KStem 算法 | 英语（较温和） |
| [stemmer_override](stemmer-override/) | 自定义词干规则 | 覆盖默认行为 |
| [keyword_marker](keyword-marker/) | 标记不做词干处理的词 | 保护特定词汇 |

> **选型建议**：英语用 `stemmer` + `english`，需要更精确控制用 `kstem`

### 停用词与过滤

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [stop](stop/) | 移除停用词 | 过滤 the, a, is 等 |
| [keep_words](keep-words/) | 仅保留指定词 | 白名单过滤 |
| [keep_types](keep-types/) | 按类型过滤 | 保留/移除数字等 |
| [length](length/) | 按长度过滤 | 移除过短/过长词元 |
| [limit](limit/) | 限制词元数量 | 防止超大文档 |

### 同义词扩展

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [synonym](synonym/) | 同义词替换/扩展 | 基础同义词 |
| [synonym_graph](synonym-graph/) | 支持多词同义词 | **推荐**，更灵活 |

### N-gram 与部分匹配

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [ngram](n-gram/) | 生成 n-gram | 模糊匹配 |
| [edge_ngram](edge-n-gram/) | 边缘 n-gram | 自动补全 |
| [shingle](shingle/) | 词组组合 | 短语匹配优化 |

### 中日韩（CJK）专用

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [cjk_bigram](cjk-bigram/) | CJK 二元分词 | 中日韩基础分词 |
| [cjk_width](cjk-width/) | 全角/半角转换 | 字符标准化 |

### 语言特定归一化

| 过滤器 | 说明 |
|--------|------|
| [arabic_normalization](arabic-normalization/) | 阿拉伯语归一化 |
| [bengali_normalization](bengali-normalization/) | 孟加拉语归一化 |
| [german_normalization](german-normalization/) | 德语归一化 |
| [hindi_normalization](hindi-normalization/) | 印地语归一化 |
| [indic_normalization](indic-normalization/) | 印度语系通用归一化 |
| [persian_normalization](persian-normalization/) | 波斯语归一化 |
| [sorani_normalization](sorani-normalization/) | 索拉尼语归一化 |
| [scandinavian_normalization](scandinavian-normalization/) | 斯堪的纳维亚语归一化 |
| [scandinavian_folding](scandinavian-folding/) | 斯堪的纳维亚字符折叠 |
| [serbian_normalization](serbian-normalization/) | 塞尔维亚语归一化 |

### 模式处理

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [pattern_capture](pattern-capture/) | 正则捕获 | 提取模式 |
| [pattern_replace](pattern-replace/) | 正则替换 | 词元转换 |

### 复合词处理

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [dictionary_decompounder](dictionary-decompounder/) | 词典拆分复合词 | 德语等复合词语言 |
| [word_delimiter](word-delimiter/) | 按分隔符拆分 | `WiFi` → `Wi`, `Fi` |
| [word_delimiter_graph](word-delimiter-graph/) | 增强版分隔符拆分 | **推荐**，支持图结构 |

### 词元增强

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [common_grams](common-grams/) | 常用词二元组 | 短语查询优化 |
| [fingerprint](fingerprint/) | 排序去重连接 | 文档指纹 |
| [multiplexer](multiplexer/) | 多路复用 | 同位置多词元 |

### 工具类过滤器

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [trim](trim/) | 去除首尾空白 | 清理词元 |
| [truncate](truncate/) | 截断过长词元 | 限制词元长度 |
| [unique](unique/) | 去除重复 | 词元去重 |
| [remove_duplicates](remove-duplicates/) | 移除同位置重复 | 精确去重 |
| [reverse](reverse/) | 反转字符串 | 后缀搜索 |
| [elision](elision/) | 移除省略词 | 法语 `l'avion` → `avion` |
| [apostrophe](apostrophe/) | 移除省略符号后内容 | 土耳其语等 |
| [classic](classic/) | 经典过滤器 | 配合 classic 分词器 |
| [decimal_digit](decimal-digit/) | Unicode 数字→ASCII | 数字标准化 |

### 高级/图处理

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [flatten_graph](flatten-graph/) | 扁平化词元图 | 索引图结构输出 |
| [conditional](condition/) | 条件过滤 | 脚本控制处理 |
| [predicate_token_filter](predicate-token-filter/) | 谓词过滤 | 自定义过滤逻辑 |

### 负载与特殊用途

| 过滤器 | 说明 | 典型用途 |
|--------|------|----------|
| [delimited_payload](delimited-payload/) | 分隔符负载 | 词元附加权重 |
| [keyword_repeat](keyword-repeat/) | 关键字重复 | 保留原词+词干 |
| [min_hash](min-hash/) | 最小哈希 | 文档相似度 |

---

## 常用组合示例

### 英文全文搜索标准配置

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "english_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "english_stop",
            "english_stemmer"
          ]
        }
      },
      "filter": {
        "english_stop": {
          "type": "stop",
          "stopwords": "_english_"
        },
        "english_stemmer": {
          "type": "stemmer",
          "language": "english"
        }
      }
    }
  }
}
```

### 同义词 + 词干提取

```json
{
  "analyzer": {
    "synonym_analyzer": {
      "type": "custom",
      "tokenizer": "standard",
      "filter": [
        "lowercase",
        "my_synonyms",
        "english_stemmer"
      ]
    }
  },
  "filter": {
    "my_synonyms": {
      "type": "synonym_graph",
      "synonyms": [
        "quick, fast, speedy",
        "big, large, huge"
      ]
    }
  }
}
```

### 自动补全（edge_ngram）

```json
{
  "analyzer": {
    "autocomplete": {
      "type": "custom",
      "tokenizer": "standard",
      "filter": [
        "lowercase",
        "autocomplete_filter"
      ]
    }
  },
  "filter": {
    "autocomplete_filter": {
      "type": "edge_ngram",
      "min_gram": 1,
      "max_gram": 20
    }
  }
}
```

---

## 过滤器顺序最佳实践

过滤器顺序影响分析结果，推荐顺序：

```
1. lowercase         ← 先转小写
2. asciifolding      ← 音标折叠
3. synonym_graph     ← 同义词扩展
4. stop              ← 移除停用词
5. stemmer           ← 词干提取
```

**原因**：
- 同义词词典通常用小写，所以 `lowercase` 在前
- 停用词移除在词干提取前，避免词干后的词被误匹配
- `synonym_graph` 需要在词干提取前，否则词干无法匹配同义词

---

## 测试词元过滤器

```json
POST /_analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase", "porter_stem"],
  "text": "The QUICK brown Foxes jumped"
}
```

测试自定义过滤器链：

```json
POST /my-index/_analyze
{
  "analyzer": "english_analyzer",
  "text": "The quick brown foxes are jumping"
}
```

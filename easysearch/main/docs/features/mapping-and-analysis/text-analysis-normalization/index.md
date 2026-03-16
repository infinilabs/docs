---
title: "归一化与规范化器"
date: 0001-01-01
description: "词元归一化概念与 Normalizer 配置参考：大小写转换、变音符号处理、Unicode 规范化、keyword 字段归一化。"
summary: "归一化与规范化器 #  词元归一化是将不同形式的词项统一为标准形式的过程，这对于提高搜索的召回率非常重要。例如，搜索 rôle 时也应该能匹配到 role。
为什么需要归一化？ #  在搜索中，我们经常遇到这样的情况：
 同一个词的不同大小写形式：The、the、THE 带变音符号和不带变音符号的词：rôle、role Unicode 的不同表示形式：é 可能以不同方式编码  如果不进行归一化，这些不同的形式会被视为不同的词项，导致搜索时无法匹配。
小写转换（Lowercasing） #  小写转换是最常见的归一化操作。大多数分析器默认都会进行小写转换。
使用 lowercase 过滤器 #  PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_lowercase_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;] } } } } } 测试效果：
GET /my_index/_analyze { &#34;analyzer&#34;: &#34;my_lowercase_analyzer&#34;, &#34;text&#34;: &#34;The Quick Brown Fox&#34; } 输出：[the, quick, brown, fox]
何时需要保留大小写？ #  某些场景下可能需要保留大小写："
---


# 归一化与规范化器

词元归一化是将不同形式的词项统一为标准形式的过程，这对于提高搜索的召回率非常重要。例如，搜索 `rôle` 时也应该能匹配到 `role`。

## 为什么需要归一化？

在搜索中，我们经常遇到这样的情况：

- 同一个词的不同大小写形式：`The`、`the`、`THE`
- 带变音符号和不带变音符号的词：`rôle`、`role`
- Unicode 的不同表示形式：`é` 可能以不同方式编码

如果不进行归一化，这些不同的形式会被视为不同的词项，导致搜索时无法匹配。

## 小写转换（Lowercasing）

小写转换是最常见的归一化操作。大多数分析器默认都会进行小写转换。

### 使用 lowercase 过滤器

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_lowercase_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase"]
        }
      }
    }
  }
}
```

测试效果：

```json
GET /my_index/_analyze
{
  "analyzer": "my_lowercase_analyzer",
  "text": "The Quick Brown Fox"
}
```

输出：`[the, quick, brown, fox]`

### 何时需要保留大小写？

某些场景下可能需要保留大小写：

- **专有名词**：`iPhone`、`C++`
- **缩写**：`USA`、`NASA`
- **代码标识符**：`userId`、`getUserInfo`

对于这些场景，可以使用 `keyword` 类型或自定义分析器。

## 移除变音符号（Removing Diacritics）

变音符号（如 `é`、`ö`、`ñ`）在某些语言中很重要，但在搜索时可能需要忽略它们。

### 使用 asciifolding 过滤器

`asciifolding` 过滤器可以将带变音符号的字符转换为对应的 ASCII 字符：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_asciifolding_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  }
}
```

测试效果：

```json
GET /my_index/_analyze
{
  "analyzer": "my_asciifolding_analyzer",
  "text": "rôle café naïve"
}
```

输出：`[role, cafe, naive]`

### 保留变音符号的场景

对于某些语言，变音符号是区分词义的关键：

- **西班牙语**：`año`（年）vs `ano`（肛门）
- **德语**：`schön`（美丽）vs `schon`（已经）

在这些场景下，应该保留变音符号，或者使用多字段同时索引两种形式。

## Unicode 规范化

Unicode 中，同一个字符可能有多种表示方式：

- **组合形式**：`é` = `e` + `´`
- **预组合形式**：`é` = 单个字符 `é`

### 使用 icu_normalizer

ICU 插件提供了更强大的 Unicode 规范化功能：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_icu_normalizer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["icu_normalizer", "lowercase"]
        }
      }
    }
  }
}
```

## 字符折叠（Character Folding）

字符折叠是将相似字符统一的过程，例如：

- `™` → `tm`
- `©` → `c`
- `®` → `r`

### 使用 mapping 字符过滤器

可以创建自定义的字符映射：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "™ => tm",
            "© => c",
            "® => r"
          ]
        }
      },
      "analyzer": {
        "my_char_folding_analyzer": {
          "type": "custom",
          "char_filter": ["my_char_filter"],
          "tokenizer": "standard",
          "filter": ["lowercase"]
        }
      }
    }
  }
}
```

## 排序和校对（Sorting and Collations）

对于排序操作，可能需要考虑语言的特定规则：

- **德语**：`ä` 应该排在 `a` 和 `b` 之间
- **瑞典语**：`ö` 应该排在 `z` 之后

这些规则通常在应用层处理，而不是在索引时处理。

## 最佳实践

1. **默认使用小写转换**：大多数场景下都应该进行小写转换
2. **谨慎使用 asciifolding**：只在确实需要忽略变音符号时使用
3. **使用多字段**：如果需要同时支持精确匹配和归一化匹配，使用多字段
4. **测试效果**：使用 `_analyze` API 测试归一化效果

示例：同时支持精确匹配和归一化匹配

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          },
          "normalized": {
            "type": "text",
            "analyzer": "my_asciifolding_analyzer"
          }
        }
      }
    }
  }
}
```

---

## 规范化器（Normalizer）

对于 `keyword` 字段类型，可以使用 **Normalizer**（规范化器）在索引和查询之前对值进行归一化处理。Normalizer 类似于分析器，但只输出单个词元，不做分词。

详细配置请参阅 [规范化器参考文档]({{< relref "/docs/features/mapping-and-analysis/text-analysis/normalizers/" >}})，包括：

- [Lowercase 规范化器]({{< relref "/docs/features/mapping-and-analysis/text-analysis/normalizers/lowercase" >}}) — 内置，直接使用
- [自定义规范化器]({{< relref "/docs/features/mapping-and-analysis/text-analysis/normalizers/custom" >}}) — 组合字符过滤器和词元过滤器

---

## 小结

- 词元归一化是将不同形式的词项统一为标准形式的过程
- 小写转换是最常见的归一化操作
- `asciifolding` 可以移除变音符号，但需要谨慎使用
- Unicode 规范化可以处理字符的不同表示形式
- Normalizer 专用于 keyword 字段，只做字符级变换不分词
- 使用多字段可以同时支持精确匹配和归一化匹配

## 相关参考

- [规范化器配置参考]({{< relref "/docs/features/mapping-and-analysis/text-analysis/normalizers/" >}})
- [词干提取](./text-analysis-stemming.md)
- [停用词](./text-analysis-stopwords.md)
- [词汇识别](./text-analysis-identifying-words.md)
- [字符过滤器]({{< relref "/docs/features/mapping-and-analysis/text-analysis/character-filters/_index.md" >}})
- [词元过滤器：normalization]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/normalization.md" >}})


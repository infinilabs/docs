---
title: "字符过滤器（Character Filters）"
date: 0001-01-01
description: "字符过滤器在分词之前预处理原始文本"
summary: "字符过滤器（Character Filters） #  字符过滤器是分析链的第一道工序，在分词器处理之前对原始文本进行预处理。
 概念指南 → 理解文本预处理，请阅读 词元归一化
  字符过滤器的位置 #  原始文本 ──→ [字符过滤器] ──→ 分词器 ──→ 词元过滤器 ──→ 索引 ↑ 预处理阶段 关键区别：
 字符过滤器：处理原始字符流（分词前） 词元过滤器：处理词元流（分词后）   内置字符过滤器 #  Easysearch 提供 3 种内置字符过滤器：
   字符过滤器 说明 典型用途      html_strip 移除 HTML 标签，解码实体 网页内容索引    mapping 字符/字符串替换映射 符号标准化、特殊字符处理    pattern_replace 正则表达式替换 复杂模式清洗、格式转换    插件提供的字符过滤器 #     字符过滤器 插件 说明 典型用途      icu_normalizer analysis-icu ICU Unicode 归一化 多语言字符统一    stconvert analysis-stconvert 中文简繁体转换 简繁体互搜     字符过滤器详解 #  html_strip — HTML 清理 #  移除 HTML/XML 标签，将 HTML 实体解码为对应字符。"
---


# 字符过滤器（Character Filters）

字符过滤器是分析链的**第一道工序**，在分词器处理之前对原始文本进行预处理。

> **概念指南** → 理解文本预处理，请阅读 [词元归一化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

---

## 字符过滤器的位置

```
原始文本 ──→ [字符过滤器] ──→ 分词器 ──→ 词元过滤器 ──→ 索引
              ↑
          预处理阶段
```

**关键区别**：
- 字符过滤器：处理**原始字符流**（分词前）
- 词元过滤器：处理**词元流**（分词后）

---

## 内置字符过滤器

Easysearch 提供 3 种内置字符过滤器：

| 字符过滤器 | 说明 | 典型用途 |
|------------|------|----------|
| [html_strip](html-strip/) | 移除 HTML 标签，解码实体 | 网页内容索引 |
| [mapping](mapping/) | 字符/字符串替换映射 | 符号标准化、特殊字符处理 |
| [pattern_replace](pattern-replace/) | 正则表达式替换 | 复杂模式清洗、格式转换 |

### 插件提供的字符过滤器

| 字符过滤器 | 插件 | 说明 | 典型用途 |
|------------|------|------|----------|
| [icu_normalizer](icu-normalizer/) | analysis-icu | ICU Unicode 归一化 | 多语言字符统一 |
| [stconvert](stconvert/) | analysis-stconvert | 中文简繁体转换 | 简繁体互搜 |

---

## 字符过滤器详解

### html_strip — HTML 清理

移除 HTML/XML 标签，将 HTML 实体解码为对应字符。

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "my_html_filter": {
          "type": "html_strip",
          "escaped_tags": ["b", "i"]
        }
      },
      "analyzer": {
        "html_analyzer": {
          "type": "custom",
          "char_filter": ["my_html_filter"],
          "tokenizer": "standard"
        }
      }
    }
  }
}
```

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `escaped_tags` | 保留的标签列表 | 无（移除所有） |

**转换示例**：
- `<p>Hello</p>` → `Hello`
- `&amp;` → `&`
- `&#x4e2d;` → `中`

---

### mapping — 字符映射

根据键值对映射替换字符或字符串。

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "symbol_mapping": {
          "type": "mapping",
          "mappings": [
            "٠ => 0", "١ => 1", "٢ => 2",
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "char_filter": ["symbol_mapping"],
          "tokenizer": "standard"
        }
      }
    }
  }
}
```

| 参数 | 说明 |
|------|------|
| `mappings` | 内联映射规则数组，格式 `key => value` |
| `mappings_path` | 外部映射文件路径（相对于 config 目录） |

**常见应用**：
- 阿拉伯数字转换为西方数字
- 表情符号转换为可搜索关键词
- 货币符号标准化（`€` → `EUR`）

---

### pattern_replace — 正则替换

使用 Java 正则表达式匹配并替换文本。

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "remove_digits": {
          "type": "pattern_replace",
          "pattern": "\\d+",
          "replacement": ""
        }
      },
      "analyzer": {
        "no_digits_analyzer": {
          "type": "custom",
          "char_filter": ["remove_digits"],
          "tokenizer": "standard"
        }
      }
    }
  }
}
```

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `pattern` | Java 正则表达式 | 必填 |
| `replacement` | 替换字符串，支持 `$1` 等捕获组引用 | `""` |
| `flags` | 正则标志（如 `CASE_INSENSITIVE`） | 无 |

**高级示例** — 手机号脱敏：

```json
{
  "type": "pattern_replace",
  "pattern": "(\\d{3})\\d{4}(\\d{4})",
  "replacement": "$1****$2"
}
```

`13812345678` → `138****5678`

---

## 选型指南

| 场景 | 推荐过滤器 | 示例配置 |
|------|------------|----------|
| 索引网页内容 | `html_strip` | 移除 HTML 标签 |
| 符号标准化 | `mapping` | `© => (c)` |
| 繁简转换准备 | `mapping` | 繁体→简体映射表 |
| 数据脱敏 | `pattern_replace` | 手机号、身份证掩码 |
| 格式清洗 | `pattern_replace` | 移除特殊字符 |
| 多种符号替换 | `mapping` | 一次定义多条规则 |

---

## 组合多个字符过滤器

一个分析器可以配置多个字符过滤器，按顺序执行：

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "html_filter": {
          "type": "html_strip"
        },
        "symbol_filter": {
          "type": "mapping",
          "mappings": ["& => and", "@ => at"]
        }
      },
      "analyzer": {
        "clean_analyzer": {
          "type": "custom",
          "char_filter": ["html_filter", "symbol_filter"],
          "tokenizer": "standard"
        }
      }
    }
  }
}
```

执行顺序：`原始文本 → html_filter → symbol_filter → 分词器`

---

## 测试字符过滤器

```json
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "text": "<p>Hello &amp; World</p>"
}
```

测试自定义过滤器：

```json
POST /my-index/_analyze
{
  "analyzer": "clean_analyzer",
  "text": "<div>Price: $100 & tax</div>"
}
```

---
title: "ICU 归一化字符过滤器（ICU Normalizer）"
date: 0001-01-01
summary: "ICU Normalizer 字符过滤器 #  icu_normalizer 字符过滤器使用 ICU 库在分词之前对原始文本进行 Unicode 归一化。它可以将字符的多种编码形式统一为标准形式，确保相同的&quot;视觉字符&quot;在搜索时能够匹配。
 需要插件：此过滤器由 analysis-icu 插件提供，Easysearch 默认已集成。
 与词元过滤器版本的区别 #  ICU 插件同时提供了字符过滤器和词元过滤器两个版本：
   版本 类型名 处理阶段 适用场景     字符过滤器 icu_normalizer 分词前 需要在分词前统一字符形式   词元过滤器 icu_normalizer 分词后 只需对词元进行归一化     建议：大多数场景使用字符过滤器版本效果更好，因为归一化发生在分词之前，可以避免因字符形式不同导致的分词差异。
 参数说明 #     参数 类型 默认值 说明     name String nfkc_cf Unicode 归一化方式，见下方选项   mode String compose 归一化模式：compose（合成）或 decompose（分解）   unicode_set_filter String — 可选，ICU UnicodeSet 过滤条件，仅对匹配字符进行归一化    name 可选值 #     值 说明     nfc NFC 标准归一化（合成优先）   nfkc NFKC 兼容归一化（合成优先）   nfkc_cf 默认值。NFKC 归一化 + Case Folding（大小写折叠）    使用示例 #  基本用法（默认 nfkc_cf） #  PUT /my-icu-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_icu_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;char_filter&#34;: [&#34;icu_normalizer&#34;] } } } } } 自定义归一化方式 #  PUT /my-icu-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;char_filter&#34;: { &#34;my_icu_normalizer&#34;: { &#34;type&#34;: &#34;icu_normalizer&#34;, &#34;name&#34;: &#34;nfc&#34;, &#34;mode&#34;: &#34;compose&#34; } }, &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;char_filter&#34;: [&#34;my_icu_normalizer&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;keyword&#34;, &#34;char_filter&#34;: [&#34;icu_normalizer&#34;], &#34;text&#34;: &#34;Ⅲ ﬁ ℃&#34; } 归一化结果：ⅲ→iii、ﬁ→fi、℃→°c（NFKC 兼容分解 + case folding）。"
---


# ICU Normalizer 字符过滤器

`icu_normalizer` 字符过滤器使用 ICU 库在**分词之前**对原始文本进行 Unicode 归一化。它可以将字符的多种编码形式统一为标准形式，确保相同的"视觉字符"在搜索时能够匹配。

> **需要插件**：此过滤器由 `analysis-icu` 插件提供，Easysearch 默认已集成。

## 与词元过滤器版本的区别

ICU 插件同时提供了字符过滤器和词元过滤器两个版本：

| 版本 | 类型名 | 处理阶段 | 适用场景 |
|------|--------|---------|---------|
| **字符过滤器** | `icu_normalizer` | 分词前 | 需要在分词前统一字符形式 |
| 词元过滤器 | `icu_normalizer` | 分词后 | 只需对词元进行归一化 |

> **建议**：大多数场景使用字符过滤器版本效果更好，因为归一化发生在分词之前，可以避免因字符形式不同导致的分词差异。

## 参数说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `name` | String | `nfkc_cf` | Unicode 归一化方式，见下方选项 |
| `mode` | String | `compose` | 归一化模式：`compose`（合成）或 `decompose`（分解） |
| `unicode_set_filter` | String | — | 可选，ICU UnicodeSet 过滤条件，仅对匹配字符进行归一化 |

### `name` 可选值

| 值 | 说明 |
|----|------|
| `nfc` | NFC 标准归一化（合成优先） |
| `nfkc` | NFKC 兼容归一化（合成优先） |
| `nfkc_cf` | **默认值**。NFKC 归一化 + Case Folding（大小写折叠） |

## 使用示例

### 基本用法（默认 nfkc_cf）

```json
PUT /my-icu-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_icu_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": ["icu_normalizer"]
        }
      }
    }
  }
}
```

### 自定义归一化方式

```json
PUT /my-icu-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "my_icu_normalizer": {
          "type": "icu_normalizer",
          "name": "nfc",
          "mode": "compose"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": ["my_icu_normalizer"]
        }
      }
    }
  }
}
```

### 测试效果

```json
GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": ["icu_normalizer"],
  "text": "Ⅲ ﬁ ℃"
}
```

归一化结果：`ⅲ`→`iii`、`ﬁ`→`fi`、`℃`→`°c`（NFKC 兼容分解 + case folding）。

## 典型归一化效果

| 输入 | nfkc_cf 输出 | 说明 |
|------|-------------|------|
| `Ⅲ` | `iii` | 罗马数字分解 |
| `ﬁ` | `fi` | 连字分解 |
| `Ａ` | `a` | 全角→半角 + 小写 |
| `é`（2 码点） | `é`（1 码点） | 组合字符合成 |
| `㎞` | `km` | 兼容字符分解 |

## 适用场景

- **多语言索引**：统一不同 Unicode 编码的相同字符
- **全角/半角混合**：日语、中文文本中的全角字母和数字
- **连字与特殊符号**：学术文本中的 ﬁ、ﬂ、℃ 等
- **组合字符**：变音符号的合成/分解统一


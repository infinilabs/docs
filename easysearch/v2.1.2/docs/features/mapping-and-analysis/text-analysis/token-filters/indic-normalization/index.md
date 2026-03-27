---
title: "印度语系归一化过滤器（Indic Normalization）"
date: 0001-01-01
summary: "印度语系归一化过滤器 #  indic_normalization 词元过滤器对印度语系（Indic）文本进行 Unicode 归一化，统一各印度语系脚本中字符的多种表示形式。它是孟加拉语、印地语等语言归一化的基础层。
归一化规则 #     处理 说明     Unicode 分解与合成 将组合字符序列转为标准的预组合形式（NFC 归一化）   零宽字符清理 移除零宽连接符（ZWJ）和零宽非连接符（ZWNJ）   Nukta 统一 将 base + nukta 两码点序列合并为等价的单码点字符   变体编码统一 统一不同 Unicode 编码表示的相同字符    使用示例 #  PUT my-indic-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;indic_norm&#34;: { &#34;type&#34;: &#34;indic_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_indic&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;indic_norm&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;indic_normalization&#34;], &#34;text&#34;: &#34;हिन्दी&#34; } 参数 #  此过滤器不接受任何参数。"
---


# 印度语系归一化过滤器

`indic_normalization` 词元过滤器对印度语系（Indic）文本进行 Unicode 归一化，统一各印度语系脚本中字符的多种表示形式。它是孟加拉语、印地语等语言归一化的基础层。

## 归一化规则

| 处理 | 说明 |
|------|------|
| Unicode 分解与合成 | 将组合字符序列转为标准的预组合形式（NFC 归一化） |
| 零宽字符清理 | 移除零宽连接符（ZWJ）和零宽非连接符（ZWNJ） |
| Nukta 统一 | 将 base + nukta 两码点序列合并为等价的单码点字符 |
| 变体编码统一 | 统一不同 Unicode 编码表示的相同字符 |

## 使用示例

```json
PUT my-indic-index
{
  "settings": {
    "analysis": {
      "filter": {
        "indic_norm": {
          "type": "indic_normalization"
        }
      },
      "analyzer": {
        "my_indic": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "indic_norm"]
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
  "tokenizer": "standard",
  "filter": ["indic_normalization"],
  "text": "हिन्दी"
}
```

## 参数

此过滤器不接受任何参数。

## 使用建议

`indic_normalization` 通常作为语言专用归一化的**前置步骤**，与语言专用过滤器配合使用：

| 组合 | 分析链 |
|------|--------|
| 孟加拉语 | `indic_normalization` → [`bengali_normalization`]({{< relref "./bengali-normalization" >}}) |
| 印地语 | `indic_normalization` → [`hindi_normalization`]({{< relref "./hindi-normalization" >}}) |

典型分析链顺序：

`standard` 分词器 → `lowercase` → `decimal_digit` → **`indic_normalization`** → 语言专用归一化 → `stop` → `stemmer`


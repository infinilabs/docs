---
title: "法语词干过滤器（French Stemmer）"
date: 0001-01-01
summary: "法语词干过滤器 #  french_light_stem 词元过滤器使用 Lucene 的轻量级法语词干算法，移除法语常见的形态后缀。
功能说明 #  法语分析器默认使用轻量级词干提取（light_french），而非 Snowball 算法。轻量级算法更保守：
   算法 说明 适用场景     light_french 轻量级，只移除明显后缀 默认推荐   french Snowball 完整词干 更激进的归约   minimal_french 最小化词干 最保守    使用示例 #  PUT my-french-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;fr_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;light_french&#34; } }, &#34;analyzer&#34;: { &#34;my_french&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;elision&#34;, &#34;fr_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;french&#34;, &#34;text&#34;: &#34;programmation programmeurs programmes&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language light_french / french / minimal_french 选择词干算法    在语言分析器中的位置 #   法语分析器 内置了 light_french 词干提取器。"
---


# 法语词干过滤器

`french_light_stem` 词元过滤器使用 Lucene 的轻量级法语词干算法，移除法语常见的形态后缀。

## 功能说明

法语分析器默认使用**轻量级词干提取**（`light_french`），而非 Snowball 算法。轻量级算法更保守：

| 算法 | 说明 | 适用场景 |
|------|------|---------|
| `light_french` | 轻量级，只移除明显后缀 | 默认推荐 |
| `french` | Snowball 完整词干 | 更激进的归约 |
| `minimal_french` | 最小化词干 | 最保守 |

## 使用示例

```json
PUT my-french-index
{
  "settings": {
    "analysis": {
      "filter": {
        "fr_stem": {
          "type": "stemmer",
          "language": "light_french"
        }
      },
      "analyzer": {
        "my_french": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "elision", "fr_stem"]
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
  "analyzer": "french",
  "text": "programmation programmeurs programmes"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `light_french` / `french` / `minimal_french` | 选择词干算法 |

## 在语言分析器中的位置

[法语分析器]({{< relref "../analyzers/french-analyzer" >}}) 内置了 `light_french` 词干提取器。



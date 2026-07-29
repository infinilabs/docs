---
title: "德语词干过滤器（German Stemmer）"
date: 0001-01-01
summary: "德语词干过滤器 #  german_light_stem 词元过滤器使用轻量级算法对德语文本进行词干提取。
功能说明 #  Easysearch 提供多种德语词干算法：
   算法 language 值 说明     轻量级 light_german 默认，最保守   最小化 minimal_german 只处理复数   Snowball german 标准 Snowball 算法   Snowball2 german2 改进的 Snowball    使用示例 #  PUT my-german-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;de_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;light_german&#34; } }, &#34;analyzer&#34;: { &#34;my_german&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;german_normalization&#34;, &#34;de_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;german&#34;, &#34;text&#34;: &#34;Programmierung Programmierer Programme&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language light_german / minimal_german / german / german2 选择词干算法    在语言分析器中的位置 #   德语分析器 内置了 light_german 词干提取器。"
---


# 德语词干过滤器

`german_light_stem` 词元过滤器使用轻量级算法对德语文本进行词干提取。

## 功能说明

Easysearch 提供多种德语词干算法：

| 算法 | `language` 值 | 说明 |
|------|--------------|------|
| 轻量级 | `light_german` | 默认，最保守 |
| 最小化 | `minimal_german` | 只处理复数 |
| Snowball | `german` | 标准 Snowball 算法 |
| Snowball2 | `german2` | 改进的 Snowball |

## 使用示例

```json
PUT my-german-index
{
  "settings": {
    "analysis": {
      "filter": {
        "de_stem": {
          "type": "stemmer",
          "language": "light_german"
        }
      },
      "analyzer": {
        "my_german": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "german_normalization", "de_stem"]
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
  "analyzer": "german",
  "text": "Programmierung Programmierer Programme"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `light_german` / `minimal_german` / `german` / `german2` | 选择词干算法 |

## 在语言分析器中的位置

[德语分析器]({{< relref "../analyzers/german-analyzer" >}}) 内置了 `light_german` 词干提取器。



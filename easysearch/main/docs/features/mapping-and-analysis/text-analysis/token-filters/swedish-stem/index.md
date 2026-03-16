---
title: "瑞典语词干过滤器（Swedish Stemmer）"
date: 0001-01-01
summary: "瑞典语词干过滤器 #  swedish_stemmer 词元过滤器使用 Snowball 算法对瑞典语文本进行词干提取。
功能说明 #  Easysearch 提供两种瑞典语词干算法：
   算法 language 值 说明     Snowball swedish 标准 Snowball 算法   轻量级 light_swedish 更保守的词干提取    使用示例 #  PUT my-swedish-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;sv_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;swedish&#34; } }, &#34;analyzer&#34;: { &#34;my_swedish&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;sv_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;swedish&#34;, &#34;text&#34;: &#34;programmering programmerare&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language swedish / light_swedish 选择词干算法    在语言分析器中的位置 #   瑞典语分析器 内置了 Snowball 词干提取器。"
---


# 瑞典语词干过滤器

`swedish_stemmer` 词元过滤器使用 Snowball 算法对瑞典语文本进行词干提取。

## 功能说明

Easysearch 提供两种瑞典语词干算法：

| 算法 | `language` 值 | 说明 |
|------|--------------|------|
| Snowball | `swedish` | 标准 Snowball 算法 |
| 轻量级 | `light_swedish` | 更保守的词干提取 |

## 使用示例

```json
PUT my-swedish-index
{
  "settings": {
    "analysis": {
      "filter": {
        "sv_stem": {
          "type": "stemmer",
          "language": "swedish"
        }
      },
      "analyzer": {
        "my_swedish": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "sv_stem"]
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
  "analyzer": "swedish",
  "text": "programmering programmerare"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `swedish` / `light_swedish` | 选择词干算法 |

## 在语言分析器中的位置

[瑞典语分析器]({{< relref "../analyzers/swedish-analyzer" >}}) 内置了 Snowball 词干提取器。



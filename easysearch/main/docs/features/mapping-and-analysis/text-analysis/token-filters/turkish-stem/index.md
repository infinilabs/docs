---
title: "土耳其语词干过滤器（Turkish Stemmer）"
date: 0001-01-01
summary: "土耳其语词干过滤器 #  turkish_stemmer 词元过滤器使用 Snowball 算法对土耳其语文本进行词干提取。
功能说明 #  土耳其语是黏着语，一个词可以有多层后缀。此词干提取器移除常见的名词格后缀、所有格后缀和动词变位后缀。
 注意：土耳其语有特殊的大小写规则（İ↔i、I↔ı），需要搭配 turkish_lowercase 过滤器。
 使用示例 #  PUT my-turkish-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;tr_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;turkish&#34; } }, &#34;analyzer&#34;: { &#34;my_turkish&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;apostrophe&#34;, &#34;turkish_lowercase&#34;, &#34;tr_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;turkish&#34;, &#34;text&#34;: &#34;programcıların programlama&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language turkish 指定土耳其语 Snowball 词干算法    在语言分析器中的位置 #   土耳其语分析器 内置了此过滤器，搭配 apostrophe 和 turkish_lowercase。"
---


# 土耳其语词干过滤器

`turkish_stemmer` 词元过滤器使用 Snowball 算法对土耳其语文本进行词干提取。

## 功能说明

土耳其语是黏着语，一个词可以有多层后缀。此词干提取器移除常见的名词格后缀、所有格后缀和动词变位后缀。

> **注意**：土耳其语有特殊的大小写规则（`İ`↔`i`、`I`↔`ı`），需要搭配 `turkish_lowercase` 过滤器。

## 使用示例

```json
PUT my-turkish-index
{
  "settings": {
    "analysis": {
      "filter": {
        "tr_stem": {
          "type": "stemmer",
          "language": "turkish"
        }
      },
      "analyzer": {
        "my_turkish": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["apostrophe", "turkish_lowercase", "tr_stem"]
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
  "analyzer": "turkish",
  "text": "programcıların programlama"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `turkish` | 指定土耳其语 Snowball 词干算法 |

## 在语言分析器中的位置

[土耳其语分析器]({{< relref "../analyzers/turkish-analyzer" >}}) 内置了此过滤器，搭配 `apostrophe` 和 `turkish_lowercase`。



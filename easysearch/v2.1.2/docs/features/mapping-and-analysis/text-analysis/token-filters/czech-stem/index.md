---
title: "捷克语词干过滤器（Czech Stemmer）"
date: 0001-01-01
summary: "捷克语词干过滤器 #  czech_stemmer 词元过滤器使用 Lucene 的轻量级捷克语词干算法，移除捷克语常见的形态后缀。
功能说明 #  此过滤器使用轻量级算法，适度移除名词格变化和动词变位后缀，在词干精度和召回率之间取得平衡。
使用示例 #  PUT my-czech-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;czech_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;czech&#34; } }, &#34;analyzer&#34;: { &#34;my_czech&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;czech_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;czech&#34;, &#34;text&#34;: &#34;programátorů programování&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language czech 指定捷克语词干算法    在语言分析器中的位置 #   捷克语分析器 内置了此过滤器。"
---


# 捷克语词干过滤器

`czech_stemmer` 词元过滤器使用 Lucene 的轻量级捷克语词干算法，移除捷克语常见的形态后缀。

## 功能说明

此过滤器使用轻量级算法，适度移除名词格变化和动词变位后缀，在词干精度和召回率之间取得平衡。

## 使用示例

```json
PUT my-czech-index
{
  "settings": {
    "analysis": {
      "filter": {
        "czech_stem": {
          "type": "stemmer",
          "language": "czech"
        }
      },
      "analyzer": {
        "my_czech": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "czech_stem"]
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
  "analyzer": "czech",
  "text": "programátorů programování"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `czech` | 指定捷克语词干算法 |

## 在语言分析器中的位置

[捷克语分析器]({{< relref "../analyzers/czech-analyzer" >}}) 内置了此过滤器。



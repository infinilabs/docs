---
title: "阿拉伯语词干过滤器（Arabic Stemmer）"
date: 0001-01-01
summary: "阿拉伯语词干过滤器 #  arabic_stemmer 词元过滤器使用 Lucene 的 ArabicStemmer 对阿拉伯语词元进行词干提取，去除常见的前缀和后缀。
词干规则 #  此词干提取器基于 Shereen Khoja 的轻量级方法，处理以下词缀：
   类型 示例     定冠词前缀 ال (al-)   介词前缀 و (wa-), ب (bi-), ك (ka-)   代词后缀 ها, هم, هن 等   阴性/双数/复数后缀 ة, ات, ين 等    使用示例 #  PUT my-arabic-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;arabic_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;arabic&#34; } }, &#34;analyzer&#34;: { &#34;my_arabic&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;arabic_normalization&#34;, &#34;arabic_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;arabic_normalization&#34;, &#34;stemmer&#34;], &#34;text&#34;: &#34;الكتابات&#34; } 参数 #  通过 stemmer 过滤器使用时："
---


# 阿拉伯语词干过滤器

`arabic_stemmer` 词元过滤器使用 Lucene 的 `ArabicStemmer` 对阿拉伯语词元进行词干提取，去除常见的前缀和后缀。

## 词干规则

此词干提取器基于 Shereen Khoja 的轻量级方法，处理以下词缀：

| 类型 | 示例 |
|------|------|
| 定冠词前缀 | ال (al-) |
| 介词前缀 | و (wa-), ب (bi-), ك (ka-) |
| 代词后缀 | ها, هم, هن 等 |
| 阴性/双数/复数后缀 | ة, ات, ين 等 |

## 使用示例

```json
PUT my-arabic-index
{
  "settings": {
    "analysis": {
      "filter": {
        "arabic_stem": {
          "type": "stemmer",
          "language": "arabic"
        }
      },
      "analyzer": {
        "my_arabic": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "arabic_normalization", "arabic_stem"]
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
  "filter": ["arabic_normalization", "stemmer"],
  "text": "الكتابات"
}
```

## 参数

通过 `stemmer` 过滤器使用时：

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `arabic` | 指定阿拉伯语词干算法 |

## 在语言分析器中的位置

[阿拉伯语分析器]({{< relref "../analyzers/arabic-analyzer" >}}) 内置了此过滤器，位于分析链末端。



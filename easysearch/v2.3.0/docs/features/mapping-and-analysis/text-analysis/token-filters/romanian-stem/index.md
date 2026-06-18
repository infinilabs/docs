---
title: "罗马尼亚语词干过滤器（Romanian Stemmer）"
date: 0001-01-01
summary: "罗马尼亚语词干过滤器 #  romanian_stemmer 词元过滤器使用 Snowball 算法对罗马尼亚语文本进行词干提取。
功能说明 #  此过滤器移除罗马尼亚语名词的格变化和定冠词后缀，以及动词变位后缀。
使用示例 #  PUT my-romanian-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;ro_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;romanian&#34; } }, &#34;analyzer&#34;: { &#34;my_romanian&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;ro_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;romanian&#34;, &#34;text&#34;: &#34;programarea programatori&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language romanian 指定罗马尼亚语 Snowball 词干算法    在语言分析器中的位置 #   罗马尼亚语分析器 内置了此过滤器。"
---


# 罗马尼亚语词干过滤器

`romanian_stemmer` 词元过滤器使用 Snowball 算法对罗马尼亚语文本进行词干提取。

## 功能说明

此过滤器移除罗马尼亚语名词的格变化和定冠词后缀，以及动词变位后缀。

## 使用示例

```json
PUT my-romanian-index
{
  "settings": {
    "analysis": {
      "filter": {
        "ro_stem": {
          "type": "stemmer",
          "language": "romanian"
        }
      },
      "analyzer": {
        "my_romanian": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "ro_stem"]
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
  "analyzer": "romanian",
  "text": "programarea programatori"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `romanian` | 指定罗马尼亚语 Snowball 词干算法 |

## 在语言分析器中的位置

[罗马尼亚语分析器]({{< relref "../analyzers/romanian-analyzer" >}}) 内置了此过滤器。



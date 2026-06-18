---
title: "荷兰语词干过滤器（Dutch Stemmer）"
date: 0001-01-01
summary: "荷兰语词干过滤器 #  dutch_stemmer 词元过滤器使用 Snowball 算法对荷兰语文本进行词干提取。
功能说明 #  荷兰语词干提取使用 Snowball 算法，结合词干覆盖字典处理不规则变形：
 移除常见名词/动词后缀 荷兰语分析器额外使用 stemmer_override 字典处理不规则形式 适合荷兰语和佛兰德语文本  使用示例 #  PUT my-dutch-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;dutch_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;dutch&#34; } }, &#34;analyzer&#34;: { &#34;my_dutch&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;dutch_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;dutch&#34;, &#34;text&#34;: &#34;programma&#39;s programmering&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language dutch 指定荷兰语 Snowball 词干算法    可选的 dutch_kp 变体使用 Krovetz-Porter 混合算法。"
---


# 荷兰语词干过滤器

`dutch_stemmer` 词元过滤器使用 Snowball 算法对荷兰语文本进行词干提取。

## 功能说明

荷兰语词干提取使用 Snowball 算法，结合词干覆盖字典处理不规则变形：

- 移除常见名词/动词后缀
- 荷兰语分析器额外使用 `stemmer_override` 字典处理不规则形式
- 适合荷兰语和佛兰德语文本

## 使用示例

```json
PUT my-dutch-index
{
  "settings": {
    "analysis": {
      "filter": {
        "dutch_stem": {
          "type": "stemmer",
          "language": "dutch"
        }
      },
      "analyzer": {
        "my_dutch": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "dutch_stem"]
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
  "analyzer": "dutch",
  "text": "programma's programmering"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `dutch` | 指定荷兰语 Snowball 词干算法 |

可选的 `dutch_kp` 变体使用 Krovetz-Porter 混合算法。

## 在语言分析器中的位置

[荷兰语分析器]({{< relref "../analyzers/dutch-analyzer" >}}) 内置了 Snowball 词干 + `stemmer_override` 字典。



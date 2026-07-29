---
title: "巴西葡萄牙语词干过滤器（Brazilian Stemmer）"
date: 0001-01-01
summary: "巴西葡萄牙语词干过滤器 #  brazilian_stemmer 词元过滤器对巴西葡萄牙语文本进行词干提取，使用 Lucene 的 BrazilianStemmer 算法。
功能说明 #  与通用葡萄牙语词干提取不同，此过滤器专门针对巴西葡萄牙语的特点进行优化：
 处理巴西特有的动词变位形式 移除名词/形容词的阴阳性和单复数后缀 处理副词后缀 -mente  使用示例 #  PUT my-brazilian-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;br_stem&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;brazilian&#34; } }, &#34;analyzer&#34;: { &#34;my_brazilian&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;br_stem&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;analyzer&#34;: &#34;brazilian&#34;, &#34;text&#34;: &#34;brasileiros programação&#34; } 参数 #     参数 值 说明     type stemmer 过滤器类型   language brazilian 指定巴西葡萄牙语词干算法    在语言分析器中的位置 #   巴西葡萄牙语分析器 内置了此过滤器。"
---


# 巴西葡萄牙语词干过滤器

`brazilian_stemmer` 词元过滤器对巴西葡萄牙语文本进行词干提取，使用 Lucene 的 `BrazilianStemmer` 算法。

## 功能说明

与通用葡萄牙语词干提取不同，此过滤器专门针对巴西葡萄牙语的特点进行优化：

- 处理巴西特有的动词变位形式
- 移除名词/形容词的阴阳性和单复数后缀
- 处理副词后缀 `-mente`

## 使用示例

```json
PUT my-brazilian-index
{
  "settings": {
    "analysis": {
      "filter": {
        "br_stem": {
          "type": "stemmer",
          "language": "brazilian"
        }
      },
      "analyzer": {
        "my_brazilian": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "br_stem"]
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
  "analyzer": "brazilian",
  "text": "brasileiros programação"
}
```

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `stemmer` | 过滤器类型 |
| `language` | `brazilian` | 指定巴西葡萄牙语词干算法 |

## 在语言分析器中的位置

[巴西葡萄牙语分析器]({{< relref "../analyzers/brazilian-analyzer" >}}) 内置了此过滤器。



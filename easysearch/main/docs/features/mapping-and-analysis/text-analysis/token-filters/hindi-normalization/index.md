---
title: "印地语归一化过滤器（Hindi Normalization）"
date: 0001-01-01
summary: "印地语归一化过滤器 #  hindi_normalization 词元过滤器对印地语（हिन्दी）文本进行正字法归一化，统一 Devanagari 脚本中的字符变体。
归一化规则 #     处理 说明     Nukta 移除 移除 nukta 标记（用于外来词音译的点）   视觉归一 统一视觉上相同但编码不同的字符变体   末尾 Chandra 移除 移除词尾的 chandra 符号   搭配 indic_normalization 建议先应用 indic_normalization 过滤器    使用示例 #  PUT my-hindi-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;hindi_norm&#34;: { &#34;type&#34;: &#34;hindi_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_hindi&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;indic_normalization&#34;, &#34;hindi_norm&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;indic_normalization&#34;, &#34;hindi_normalization&#34;], &#34;text&#34;: &#34;हिन्दी भाषा&#34; } 参数 #  此过滤器不接受任何参数。"
---


# 印地语归一化过滤器

`hindi_normalization` 词元过滤器对印地语（हिन्दी）文本进行正字法归一化，统一 Devanagari 脚本中的字符变体。

## 归一化规则

| 处理 | 说明 |
|------|------|
| Nukta 移除 | 移除 nukta 标记（用于外来词音译的点） |
| 视觉归一 | 统一视觉上相同但编码不同的字符变体 |
| 末尾 Chandra 移除 | 移除词尾的 chandra 符号 |
| 搭配 `indic_normalization` | 建议先应用 `indic_normalization` 过滤器 |

## 使用示例

```json
PUT my-hindi-index
{
  "settings": {
    "analysis": {
      "filter": {
        "hindi_norm": {
          "type": "hindi_normalization"
        }
      },
      "analyzer": {
        "my_hindi": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "indic_normalization", "hindi_norm"]
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
  "filter": ["indic_normalization", "hindi_normalization"],
  "text": "हिन्दी भाषा"
}
```

## 参数

此过滤器不接受任何参数。

## 在语言分析器中的位置

[印地语分析器]({{< relref "../analyzers/hindi-analyzer" >}}) 内置了此过滤器，其分析链为：

`standard` 分词器 → `lowercase` → `decimal_digit` → `indic_normalization` → **`hindi_normalization`** → `stop`（印地语） → `hindi_stemmer`



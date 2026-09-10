---
title: "孟加拉语归一化过滤器（Bengali Normalization）"
date: 0001-01-01
summary: "孟加拉语归一化过滤器 #  bengali_normalization 词元过滤器对孟加拉语（বাংলা）文本进行 Unicode 归一化，统一字符的多种表示形式。
归一化规则 #     处理 说明     Nukta 合成 将 base + nukta 组合转为对应的预组合字符   变体统一 统一视觉上相同但编码不同的字符   印度语系通用归一化 在 indic_normalization 基础上进一步处理    使用示例 #  PUT my-bengali-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;bengali_norm&#34;: { &#34;type&#34;: &#34;bengali_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_bengali&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;indic_normalization&#34;, &#34;bengali_norm&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;bengali_normalization&#34;], &#34;text&#34;: &#34;বাংলাদেশ&#34; } 参数 #  此过滤器不接受任何参数。"
---


# 孟加拉语归一化过滤器

`bengali_normalization` 词元过滤器对孟加拉语（বাংলা）文本进行 Unicode 归一化，统一字符的多种表示形式。

## 归一化规则

| 处理 | 说明 |
|------|------|
| Nukta 合成 | 将 base + nukta 组合转为对应的预组合字符 |
| 变体统一 | 统一视觉上相同但编码不同的字符 |
| 印度语系通用归一化 | 在 `indic_normalization` 基础上进一步处理 |

## 使用示例

```json
PUT my-bengali-index
{
  "settings": {
    "analysis": {
      "filter": {
        "bengali_norm": {
          "type": "bengali_normalization"
        }
      },
      "analyzer": {
        "my_bengali": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "indic_normalization", "bengali_norm"]
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
  "filter": ["bengali_normalization"],
  "text": "বাংলাদেশ"
}
```

## 参数

此过滤器不接受任何参数。

## 在语言分析器中的位置

[孟加拉语分析器]({{< relref "../analyzers/bengali-analyzer" >}}) 内置了此过滤器，其分析链为：

`standard` 分词器 → `lowercase` → `decimal_digit` → `indic_normalization` → **`bengali_normalization`** → `stop`（孟加拉语） → `bengali_stemmer`



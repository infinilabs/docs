---
title: "索拉尼语归一化过滤器（Sorani Normalization）"
date: 0001-01-01
summary: "索拉尼语归一化过滤器 #  sorani_normalization 词元过滤器对索拉尼库尔德语（سۆرانی）文本进行字符归一化。索拉尼语使用修改后的阿拉伯字母书写。
归一化规则 #     处理 说明     Yaa 归一化 统一 ي/ی 变体   Kaf 归一化 统一 ك/ک 变体   Haa 归一化 统一 haa 变体   变音符号移除 移除可选的变音标记    使用示例 #  PUT my-sorani-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;sorani_norm&#34;: { &#34;type&#34;: &#34;sorani_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_sorani&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;sorani_norm&#34;, &#34;sorani_stemmer&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;sorani_normalization&#34;], &#34;text&#34;: &#34;کوردی&#34; } 参数 #  此过滤器不接受任何参数。"
---


# 索拉尼语归一化过滤器

`sorani_normalization` 词元过滤器对索拉尼库尔德语（سۆرانی）文本进行字符归一化。索拉尼语使用修改后的阿拉伯字母书写。

## 归一化规则

| 处理 | 说明 |
|------|------|
| Yaa 归一化 | 统一 ي/ی 变体 |
| Kaf 归一化 | 统一 ك/ک 变体 |
| Haa 归一化 | 统一 haa 变体 |
| 变音符号移除 | 移除可选的变音标记 |

## 使用示例

```json
PUT my-sorani-index
{
  "settings": {
    "analysis": {
      "filter": {
        "sorani_norm": {
          "type": "sorani_normalization"
        }
      },
      "analyzer": {
        "my_sorani": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "sorani_norm", "sorani_stemmer"]
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
  "filter": ["sorani_normalization"],
  "text": "کوردی"
}
```

## 参数

此过滤器不接受任何参数。

## 在语言分析器中的位置

[索拉尼语分析器]({{< relref "../analyzers/sorani-analyzer" >}}) 内置了此过滤器，其分析链为：

`standard` 分词器 → `lowercase` → `decimal_digit` → **`sorani_normalization`** → `stop`（索拉尼语） → `sorani_stemmer`



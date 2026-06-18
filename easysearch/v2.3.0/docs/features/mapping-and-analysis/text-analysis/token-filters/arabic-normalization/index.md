---
title: "阿拉伯语归一化过滤器（Arabic Normalization）"
date: 0001-01-01
summary: "阿拉伯语归一化过滤器 #  arabic_normalization 词元过滤器对阿拉伯语文本进行正字法归一化，将各种书写变体统一为标准形式，提高搜索的召回率。
归一化规则 #     原始形式 归一化结果 说明     \u0623 \u0625 \u0622 (带 hamza 的 alef) \u0627 (bare alef) 统一 alef 变体   \u0629 (taa marbuta) \u0647 (haa) 统一词尾形式   \u064e \u064f \u0650 \u064b \u064c \u064d (harakat) 删除 移除变音符号   \u0640 (tatweel/kashida) 删除 移除装饰性延长符    使用示例 #  PUT my-arabic-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;arabic_norm&#34;: { &#34;type&#34;: &#34;arabic_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_arabic&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;arabic_norm&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;arabic_normalization&#34;], &#34;text&#34;: &#34;\u0625\u0628\u0631\u0627\u0647\u064a\u0645&#34; } 参数 #  此过滤器不接受任何参数。"
---


# 阿拉伯语归一化过滤器

`arabic_normalization` 词元过滤器对阿拉伯语文本进行正字法归一化，将各种书写变体统一为标准形式，提高搜索的召回率。

## 归一化规则

| 原始形式 | 归一化结果 | 说明 |
|---------|-----------|------|
| `\u0623` `\u0625` `\u0622` (带 hamza 的 alef) | `\u0627` (bare alef) | 统一 alef 变体 |
| `\u0629` (taa marbuta) | `\u0647` (haa) | 统一词尾形式 |
| `\u064e` `\u064f` `\u0650` `\u064b` `\u064c` `\u064d` (harakat) | 删除 | 移除变音符号 |
| `\u0640` (tatweel/kashida) | 删除 | 移除装饰性延长符 |

## 使用示例

```json
PUT my-arabic-index
{
  "settings": {
    "analysis": {
      "filter": {
        "arabic_norm": {
          "type": "arabic_normalization"
        }
      },
      "analyzer": {
        "my_arabic": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "arabic_norm"]
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
  "filter": ["arabic_normalization"],
  "text": "\u0625\u0628\u0631\u0627\u0647\u064a\u0645"
}
```

## 参数

此过滤器不接受任何参数。

## 在语言分析器中的位置

[阿拉伯语分析器]({{< relref "../analyzers/arabic-analyzer" >}}) 内置了此过滤器，其分析链为：

`standard` 分词器 → `lowercase` → **`arabic_normalization`** → `decimal_digit` → `stop`（阿拉伯语） → `arabic_stemmer`



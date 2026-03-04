---
title: "波斯语归一化过滤器（Persian Normalization）"
date: 0001-01-01
summary: "波斯语归一化过滤器 #  persian_normalization 词元过滤器对波斯语（فارسی）文本进行字符归一化，统一阿拉伯语和波斯语书写变体。
归一化规则 #     处理 说明     阿拉伯语 Yaa → 波斯语 Yaa 统一 ي → ی   阿拉伯语 Kaf → 波斯语 Kaf 统一 ك → ک   变音符号移除 移除阿拉伯语 harakat 标记   搭配 arabic_normalization 建议与 arabic_normalization 一起使用    使用示例 #  PUT my-persian-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;persian_norm&#34;: { &#34;type&#34;: &#34;persian_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_persian&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;arabic_normalization&#34;, &#34;persian_norm&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;arabic_normalization&#34;, &#34;persian_normalization&#34;], &#34;text&#34;: &#34;فارسی&#34; } 参数 #  此过滤器不接受任何参数。"
---


# 波斯语归一化过滤器

`persian_normalization` 词元过滤器对波斯语（فارسی）文本进行字符归一化，统一阿拉伯语和波斯语书写变体。

## 归一化规则

| 处理 | 说明 |
|------|------|
| 阿拉伯语 Yaa → 波斯语 Yaa | 统一 ي → ی |
| 阿拉伯语 Kaf → 波斯语 Kaf | 统一 ك → ک |
| 变音符号移除 | 移除阿拉伯语 harakat 标记 |
| 搭配 `arabic_normalization` | 建议与 `arabic_normalization` 一起使用 |

## 使用示例

```json
PUT my-persian-index
{
  "settings": {
    "analysis": {
      "filter": {
        "persian_norm": {
          "type": "persian_normalization"
        }
      },
      "analyzer": {
        "my_persian": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "arabic_normalization", "persian_norm"]
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
  "filter": ["arabic_normalization", "persian_normalization"],
  "text": "فارسی"
}
```

## 参数

此过滤器不接受任何参数。

## 在语言分析器中的位置

[波斯语分析器]({{< relref "../analyzers/persian-analyzer" >}}) 内置了此过滤器，其分析链为：

`mapping` 字符过滤器（零宽非连接符）→ `standard` 分词器 → `lowercase` → `decimal_digit` → `arabic_normalization` → **`persian_normalization`** → `stop`（波斯语）



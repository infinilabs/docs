---
title: "德语归一化过滤器（German Normalization）"
date: 0001-01-01
summary: "德语归一化过滤器 #  german_normalization 词元过滤器将德语特有的元音变音和 ß 归一化为 ASCII 等价形式。
归一化规则 #     原始 归一化 说明     ä a 元音变音归一化   ö o 元音变音归一化   ü u 元音变音归一化   Ä A 大写变音归一化   Ö O 大写变音归一化   Ü U 大写变音归一化   ß ss 双 s 替换     注意：这与 ae → a 不同。此过滤器只处理变音符号字符本身，不处理 ae/oe/ue 的拼写变体。
 使用示例 #  PUT my-german-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;german_norm&#34;: { &#34;type&#34;: &#34;german_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_german&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;german_norm&#34;, &#34;german_stemmer&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;german_normalization&#34;], &#34;text&#34;: &#34;Straße Übung Ärger&#34; } 响应中 Straße → strasse、Übung → ubung、Ärger → arger。"
---


# 德语归一化过滤器

`german_normalization` 词元过滤器将德语特有的元音变音和 ß 归一化为 ASCII 等价形式。

## 归一化规则

| 原始 | 归一化 | 说明 |
|------|--------|------|
| ä | a | 元音变音归一化 |
| ö | o | 元音变音归一化 |
| ü | u | 元音变音归一化 |
| Ä | A | 大写变音归一化 |
| Ö | O | 大写变音归一化 |
| Ü | U | 大写变音归一化 |
| ß | ss | 双 s 替换 |

> 注意：这与 `ae → a` 不同。此过滤器只处理变音符号字符本身，不处理 `ae`/`oe`/`ue` 的拼写变体。

## 使用示例

```json
PUT my-german-index
{
  "settings": {
    "analysis": {
      "filter": {
        "german_norm": {
          "type": "german_normalization"
        }
      },
      "analyzer": {
        "my_german": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "german_norm", "german_stemmer"]
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
  "filter": ["german_normalization"],
  "text": "Straße Übung Ärger"
}
```

响应中 `Straße` → `strasse`、`Übung` → `ubung`、`Ärger` → `arger`。

## 参数

此过滤器不接受任何参数。

## 在语言分析器中的位置

[德语分析器]({{< relref "../analyzers/german-analyzer" >}}) 内置了此过滤器，其分析链为：

`standard` 分词器 → `lowercase` → `stop`（德语） → `keyword_marker`（stem_exclusion） → **`german_normalization`** → `german_light_stem`



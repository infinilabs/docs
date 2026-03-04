---
title: "斯堪的纳维亚归一化过滤器（Scandinavian Normalization）"
date: 0001-01-01
summary: "斯堪的纳维亚归一化过滤器 #  scandinavian_normalization 词元过滤器将斯堪的纳维亚语言中互换使用的字符对统一为一种形式，使跨语言搜索更一致。
归一化规则 #     原始字符 归一化为 说明     ä æ 瑞典语 ä → 丹麦语/挪威语 æ   ö ø 瑞典语 ö → 丹麦语/挪威语 ø   Ä Æ 大写同理   Ö Ø 大写同理     与 scandinavian_folding 的区别：scandinavian_normalization 只统一互换字符对（ä↔æ, ö↔ø），不会折叠到 ASCII 基础字符。而 scandinavian_folding 会进一步折叠为 a、o 等 ASCII 字符。
 使用示例 #  PUT my-scandi-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;scandi_norm&#34;: { &#34;type&#34;: &#34;scandinavian_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_scandi&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;scandi_norm&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;scandinavian_normalization&#34;], &#34;text&#34;: &#34;räksmörgås&#34; } 响应中 räksmörgås → pair ræksmørgås（ä→æ, ö→ø，但 å 保持不变）。"
---


# 斯堪的纳维亚归一化过滤器

`scandinavian_normalization` 词元过滤器将斯堪的纳维亚语言中互换使用的字符对统一为一种形式，使跨语言搜索更一致。

## 归一化规则

| 原始字符 | 归一化为 | 说明 |
|---------|---------|------|
| ä | æ | 瑞典语 ä → 丹麦语/挪威语 æ |
| ö | ø | 瑞典语 ö → 丹麦语/挪威语 ø |
| Ä | Æ | 大写同理 |
| Ö | Ø | 大写同理 |

> **与 `scandinavian_folding` 的区别**：`scandinavian_normalization` 只统一互换字符对（ä↔æ, ö↔ø），**不会**折叠到 ASCII 基础字符。而 [`scandinavian_folding`]({{< relref "./scandinavian-folding" >}}) 会进一步折叠为 a、o 等 ASCII 字符。

## 使用示例

```json
PUT my-scandi-index
{
  "settings": {
    "analysis": {
      "filter": {
        "scandi_norm": {
          "type": "scandinavian_normalization"
        }
      },
      "analyzer": {
        "my_scandi": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "scandi_norm"]
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
  "filter": ["scandinavian_normalization"],
  "text": "räksmörgås"
}
```

响应中 `räksmörgås` → `pair ræksmørgås`（ä→æ, ö→ø，但 å 保持不变）。

## 参数

此过滤器不接受任何参数。

## 适用场景

- 跨斯堪的纳维亚语言搜索（瑞典语+丹麦语+挪威语混合索引）
- 希望统一 ä/æ 和 ö/ø，但**保留** å、ø 等字符不折叠到 ASCII
- 如果需要更激进的折叠（å→a），请改用 [`scandinavian_folding`]({{< relref "./scandinavian-folding" >}})


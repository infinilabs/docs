---
title: "斯堪的纳维亚字符折叠过滤器（Scandinavian Folding）"
date: 0001-01-01
summary: "斯堪的纳维亚字符折叠过滤器 #  scandinavian_folding 词元过滤器将斯堪的纳维亚语言中互换使用的字符统一归一化。
折叠规则 #     原始字符 折叠为 涉及语言     å a 瑞典语、丹麦语、挪威语   ä, æ a 瑞典语(ä) / 丹麦语、挪威语(æ)   ö, ø o 瑞典语(ö) / 丹麦语、挪威语(ø)     与 scandinavian_normalization 的区别：scandinavian_normalization 只折叠互换字符对（如 ä↔æ），而 scandinavian_folding 还会进一步折叠到 ASCII 基础字符（如 å→a）。
 使用示例 #  PUT my-scandi-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;scandi_fold&#34;: { &#34;type&#34;: &#34;scandinavian_folding&#34; } }, &#34;analyzer&#34;: { &#34;my_scandi&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;scandi_fold&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;scandinavian_folding&#34;], &#34;text&#34;: &#34;räksmörgås&#34; } 响应中 räksmörgås → raksmørgas 或 raksmorgas（取决于折叠层级）。"
---


# 斯堪的纳维亚字符折叠过滤器

`scandinavian_folding` 词元过滤器将斯堪的纳维亚语言中互换使用的字符统一归一化。

## 折叠规则

| 原始字符 | 折叠为 | 涉及语言 |
|---------|--------|---------|
| å | a | 瑞典语、丹麦语、挪威语 |
| ä, æ | a | 瑞典语(ä) / 丹麦语、挪威语(æ) |
| ö, ø | o | 瑞典语(ö) / 丹麦语、挪威语(ø) |

> **与 `scandinavian_normalization` 的区别**：`scandinavian_normalization` 只折叠互换字符对（如 ä↔æ），而 `scandinavian_folding` 还会进一步折叠到 ASCII 基础字符（如 å→a）。

## 使用示例

```json
PUT my-scandi-index
{
  "settings": {
    "analysis": {
      "filter": {
        "scandi_fold": {
          "type": "scandinavian_folding"
        }
      },
      "analyzer": {
        "my_scandi": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "scandi_fold"]
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
  "filter": ["scandinavian_folding"],
  "text": "räksmörgås"
}
```

响应中 `räksmörgås` → `raksmørgas` 或 `raksmorgas`（取决于折叠层级）。

## 参数

此过滤器不接受任何参数。

## 适用场景

- 跨斯堪的纳维亚语言搜索（瑞典语+丹麦语+挪威语混合索引）
- 用户可能混用 ö/ø 或 ä/æ 的场景



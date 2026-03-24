---
title: "泰语分词器（Thai）"
date: 0001-01-01
summary: "Thai 分词器 #  thai 分词器使用 Java 内置的泰语分词算法（BreakIterator）对泰语文本进行分词。对于非泰语文本，其行为与 standard 分词器相同。
示例 #  POST _analyze { &#34;tokenizer&#34;: &#34;thai&#34;, &#34;text&#34;: &#34;การทดสอบ&#34; } 相关指南 #    Standard 分词器  文本分析基础  "
---


# Thai 分词器

`thai` 分词器使用 Java 内置的泰语分词算法（`BreakIterator`）对泰语文本进行分词。对于非泰语文本，其行为与 `standard` 分词器相同。

## 示例

```json
POST _analyze
{
  "tokenizer": "thai",
  "text": "การทดสอบ"
}
```

## 相关指南

- [Standard 分词器]({{< relref "./standard" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


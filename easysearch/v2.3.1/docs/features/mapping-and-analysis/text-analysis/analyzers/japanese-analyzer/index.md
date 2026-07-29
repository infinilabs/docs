---
title: "日语分析器（Japanese）"
date: 0001-01-01
summary: "Japanese 分析器 #  japanese 分析器是为日语文本设计的基础语言分析器，使用 CJK 二元组分词方式。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 cjk 分词器：将 CJK（中日韩）字符分解为二元组（bigrams） lowercase 分词过滤器：转换为小写 cjk_width 分词过滤器：全角/半角字符归一化  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;japanese&#34;, &#34;text&#34;: &#34;東京都の天気は晴れです&#34; } 相关指南 #    语言分析器  文本分析基础  "
---


# Japanese 分析器

`japanese` 分析器是为日语文本设计的基础语言分析器，使用 CJK 二元组分词方式。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `cjk` 分词器：将 CJK（中日韩）字符分解为二元组（bigrams）
- `lowercase` 分词过滤器：转换为小写
- `cjk_width` 分词过滤器：全角/半角字符归一化

## 示例

```json
POST _analyze
{
  "analyzer": "japanese",
  "text": "東京都の天気は晴れです"
}
```

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


---
title: "泰语分析器（Thai）"
date: 0001-01-01
summary: "Thai 分析器 #  thai 分析器是为泰语文本特别设计的语言分析器，使用 Java 内置的泰语分词算法。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 thai 分词器：使用 Java BreakIterator 进行泰语分词 lowercase 分词过滤器：转换为小写 decimal_digit 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字 stop 分词过滤器：过滤泰语停用词  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;thai&#34;, &#34;text&#34;: &#34;สุนัขวิ่งในสวน&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _thai_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Thai 分析器

`thai` 分析器是为泰语文本特别设计的语言分析器，使用 Java 内置的泰语分词算法。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `thai` 分词器：使用 Java BreakIterator 进行泰语分词
- `lowercase` 分词过滤器：转换为小写
- `decimal_digit` 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字
- `stop` 分词过滤器：过滤泰语停用词

## 示例

```json
POST _analyze
{
  "analyzer": "thai",
  "text": "สุนัขวิ่งในสวน"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_thai_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


---
title: "英语分析器（English）"
date: 0001-01-01
summary: "English 分析器 #  english 分析器是为英语文本特别设计的语言分析器，包含了英语特定的处理规则和停用词。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤英语停用词（the, a, an, and 等） porter_stem 分词过滤器：英语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;english&#34;, &#34;text&#34;: &#34;The quick brown foxes jumping over the lazy dogs&#34; } 分析结果 #  停用词（the, over 等）被移除，词根被提取：
[ &#34;quick&#34;, &#34;brown&#34;, &#34;fox&#34;, &#34;jump&#34;, &#34;lazi&#34;, &#34;dog&#34; ] 相关指南 #    文本分析：词干提取  文本分析基础  "
---


# English 分析器

`english` 分析器是为英语文本特别设计的语言分析器，包含了英语特定的处理规则和停用词。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤英语停用词（the, a, an, and 等）
- `porter_stem` 分词过滤器：英语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "english",
  "text": "The quick brown foxes jumping over the lazy dogs"
}
```

## 分析结果

停用词（the, over 等）被移除，词根被提取：

```
[ "quick", "brown", "fox", "jump", "lazi", "dog" ]
```

## 相关指南

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


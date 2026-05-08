---
title: "希腊语分析器（Greek）"
date: 0001-01-01
summary: "Greek 分析器 #  greek 分析器是为希腊语文本特别设计的语言分析器，使用专用的希腊语小写转换。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 greek_lowercase 分词过滤器：希腊语专用小写转换 stop 分词过滤器：过滤希腊语停用词 greek_stem 分词过滤器：希腊语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;greek&#34;, &#34;text&#34;: &#34;Τα σκυλιά τρέχουν στο πάρκο&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _greek_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Greek 分析器

`greek` 分析器是为希腊语文本特别设计的语言分析器，使用专用的希腊语小写转换。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `greek_lowercase` 分词过滤器：希腊语专用小写转换
- `stop` 分词过滤器：过滤希腊语停用词
- `greek_stem` 分词过滤器：希腊语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "greek",
  "text": "Τα σκυλιά τρέχουν στο πάρκο"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_greek_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


---
title: "德语分析器（German）"
date: 0001-01-01
summary: "German 分析器 #  german 分析器是为德语文本特别设计的语言分析器，包含德语归一化和轻量词干提取。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤德语停用词 german_normalization 分词过滤器：德语字符归一化（ä→a, ö→o, ü→u, ß→ss） german_light_stem 分词过滤器：德语轻量词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;german&#34;, &#34;text&#34;: &#34;Die Hunde laufen im Park&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _german_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# German 分析器

`german` 分析器是为德语文本特别设计的语言分析器，包含德语归一化和轻量词干提取。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤德语停用词
- `german_normalization` 分词过滤器：德语字符归一化（ä→a, ö→o, ü→u, ß→ss）
- `german_light_stem` 分词过滤器：德语轻量词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "german",
  "text": "Die Hunde laufen im Park"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_german_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


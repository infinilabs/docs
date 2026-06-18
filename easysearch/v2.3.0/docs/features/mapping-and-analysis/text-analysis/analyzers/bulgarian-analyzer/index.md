---
title: "保加利亚语分析器（Bulgarian）"
date: 0001-01-01
summary: "Bulgarian 分析器 #  bulgarian 分析器是为保加利亚语文本特别设计的语言分析器。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤保加利亚语停用词 bulgarian_stem 分词过滤器：保加利亚语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;bulgarian&#34;, &#34;text&#34;: &#34;Кучетата тичат в парка&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _bulgarian_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Bulgarian 分析器

`bulgarian` 分析器是为保加利亚语文本特别设计的语言分析器。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤保加利亚语停用词
- `bulgarian_stem` 分词过滤器：保加利亚语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "bulgarian",
  "text": "Кучетата тичат в парка"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_bulgarian_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


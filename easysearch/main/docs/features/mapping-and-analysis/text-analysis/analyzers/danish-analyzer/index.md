---
title: "丹麦语分析器（Danish）"
date: 0001-01-01
summary: "Danish 分析器 #  danish 分析器是为丹麦语文本特别设计的语言分析器。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤丹麦语停用词 snowball(Danish) 分词过滤器：丹麦语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;danish&#34;, &#34;text&#34;: &#34;Hundene løber i parken&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _danish_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Danish 分析器

`danish` 分析器是为丹麦语文本特别设计的语言分析器。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤丹麦语停用词
- `snowball`(Danish) 分词过滤器：丹麦语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "danish",
  "text": "Hundene løber i parken"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_danish_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


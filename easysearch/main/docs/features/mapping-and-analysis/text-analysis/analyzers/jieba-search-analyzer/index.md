---
title: "Jieba 搜索分析器（Jieba Search）"
date: 0001-01-01
summary: "Jieba Search 分析器 #  jieba_search 分析器是为中文搜索分词的分析器，使用 Jieba 分词器的搜索模式，在精确模式的基础上对长词再次切分。
需要安装 analysis-jieba 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 jieba_search 分词器：使用 Jieba 搜索模式，对长词进行二次切分以提高召回率 lowercase 分词过滤器：转换为小写  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;jieba_search&#34;, &#34;text&#34;: &#34;小明硕士毕业于中国科学院计算所&#34; } 相关指南 #    文本分析基础  "
---


# Jieba Search 分析器

`jieba_search` 分析器是为中文搜索分词的分析器，使用 Jieba 分词器的搜索模式，在精确模式的基础上对长词再次切分。

{{< hint info >}}
需要安装 `analysis-jieba` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `jieba_search` 分词器：使用 Jieba 搜索模式，对长词进行二次切分以提高召回率
- `lowercase` 分词过滤器：转换为小写

## 示例

```json
POST _analyze
{
  "analyzer": "jieba_search",
  "text": "小明硕士毕业于中国科学院计算所"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


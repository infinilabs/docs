---
title: "Jieba 索引分析器（Jieba Index）"
date: 0001-01-01
summary: "Jieba Index 分析器 #  jieba_index 分析器是为中文索引分词的分析器，使用 Jieba 分词器的索引模式。
需要安装 analysis-jieba 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 jieba_index 分词器：使用 Jieba 索引模式，适合索引时的细粒度分词 lowercase 分词过滤器：转换为小写  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;jieba_index&#34;, &#34;text&#34;: &#34;小明硕士毕业于中国科学院计算所&#34; } 相关指南 #    文本分析基础  "
---


# Jieba Index 分析器

`jieba_index` 分析器是为中文索引分词的分析器，使用 Jieba 分词器的索引模式。

{{< hint info >}}
需要安装 `analysis-jieba` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `jieba_index` 分词器：使用 Jieba 索引模式，适合索引时的细粒度分词
- `lowercase` 分词过滤器：转换为小写

## 示例

```json
POST _analyze
{
  "analyzer": "jieba_index",
  "text": "小明硕士毕业于中国科学院计算所"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


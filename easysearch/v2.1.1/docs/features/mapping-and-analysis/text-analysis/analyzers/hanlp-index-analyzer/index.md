---
title: "HanLP 索引分析器（HanLP Index）"
date: 0001-01-01
summary: "HanLP Index 分析器 #  hanlp_index 分析器是为中文索引分词的分析器，使用 HanLP 索引分词模式，会对文本进行更细粒度的切分。
需要安装 analysis-hanlp 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 hanlp_index 分词器：使用 HanLP 索引分词模式，对文本进行细粒度切分 lowercase 分词过滤器：转换为小写  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;hanlp_index&#34;, &#34;text&#34;: &#34;中华人民共和国国歌&#34; } 相关指南 #    文本分析基础  "
---


# HanLP Index 分析器

`hanlp_index` 分析器是为中文索引分词的分析器，使用 HanLP 索引分词模式，会对文本进行更细粒度的切分。

{{< hint info >}}
需要安装 `analysis-hanlp` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `hanlp_index` 分词器：使用 HanLP 索引分词模式，对文本进行细粒度切分
- `lowercase` 分词过滤器：转换为小写

## 示例

```json
POST _analyze
{
  "analyzer": "hanlp_index",
  "text": "中华人民共和国国歌"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


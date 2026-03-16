---
title: "HanLP 标准分析器（HanLP Standard）"
date: 0001-01-01
summary: "HanLP Standard 分析器 #  hanlp_standard 分析器是为中文标准分词的分析器，使用 HanLP 标准分词模式，适合大多数通用中文搜索场景。
需要安装 analysis-hanlp 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 hanlp_standard 分词器：使用 HanLP 标准分词模式 lowercase 分词过滤器：转换为小写  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;hanlp_standard&#34;, &#34;text&#34;: &#34;中华人民共和国国歌&#34; } 相关指南 #    文本分析基础  "
---


# HanLP Standard 分析器

`hanlp_standard` 分析器是为中文标准分词的分析器，使用 HanLP 标准分词模式，适合大多数通用中文搜索场景。

{{< hint info >}}
需要安装 `analysis-hanlp` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `hanlp_standard` 分词器：使用 HanLP 标准分词模式
- `lowercase` 分词过滤器：转换为小写

## 示例

```json
POST _analyze
{
  "analyzer": "hanlp_standard",
  "text": "中华人民共和国国歌"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


---
title: "HanLP CRF 分析器（HanLP CRF）"
date: 0001-01-01
summary: "HanLP CRF 分析器 #  hanlp_crf 分析器是为中文 CRF 分词的分析器，使用 HanLP CRF（条件随机场）模型进行分词，适合学术研究场景。
需要安装 analysis-hanlp 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 hanlp_crf 分词器：使用 HanLP CRF 条件随机场模型进行分词 lowercase 分词过滤器：转换为小写  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;hanlp_crf&#34;, &#34;text&#34;: &#34;中国科学院计算技术研究所&#34; } 相关指南 #    文本分析基础  "
---


# HanLP CRF 分析器

`hanlp_crf` 分析器是为中文 CRF 分词的分析器，使用 HanLP CRF（条件随机场）模型进行分词，适合学术研究场景。

{{< hint info >}}
需要安装 `analysis-hanlp` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `hanlp_crf` 分词器：使用 HanLP CRF 条件随机场模型进行分词
- `lowercase` 分词过滤器：转换为小写

## 示例

```json
POST _analyze
{
  "analyzer": "hanlp_crf",
  "text": "中国科学院计算技术研究所"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


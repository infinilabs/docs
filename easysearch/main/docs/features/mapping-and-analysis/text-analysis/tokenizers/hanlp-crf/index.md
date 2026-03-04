---
title: "HanLP CRF 分词器（HanLP CRF）"
date: 0001-01-01
summary: "HanLP CRF 分词器 #  hanlp_crf 分词器使用 HanLP CRF（条件随机场）分词模式，适合需要高精度分词的学术研究场景。
需要安装 analysis-hanlp 插件，并确保 CRF CWS 模型可用。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;hanlp_crf&#34; } } } } } 相关指南 #    HanLP 分词器  HanLP CRF 分析器  文本分析基础  "
---


# HanLP CRF 分词器

`hanlp_crf` 分词器使用 HanLP CRF（条件随机场）分词模式，适合需要高精度分词的学术研究场景。

需要安装 `analysis-hanlp` 插件，并确保 CRF CWS 模型可用。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "hanlp_crf"
        }
      }
    }
  }
}
```

## 相关指南

- [HanLP 分词器]({{< relref "./hanlp" >}})
- [HanLP CRF 分析器]({{< relref "../analyzers/hanlp-crf-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


---
title: "HanLP N 最短路分词器（HanLP N-Short）"
date: 0001-01-01
summary: "HanLP N-Short 分词器 #  hanlp_n_short 分词器使用 HanLP N 最短路径分词算法，能找到全局 N 条最短路径，适合需要高精度分词的场景。
需要安装 analysis-hanlp 插件。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;hanlp_n_short&#34; } } } } } 相关指南 #    HanLP 分词器  文本分析基础  "
---


# HanLP N-Short 分词器

`hanlp_n_short` 分词器使用 HanLP N 最短路径分词算法，能找到全局 N 条最短路径，适合需要高精度分词的场景。

需要安装 `analysis-hanlp` 插件。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "hanlp_n_short"
        }
      }
    }
  }
}
```

## 相关指南

- [HanLP 分词器]({{< relref "./hanlp" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


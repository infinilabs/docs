---
title: "HanLP 标准分词器（HanLP Standard）"
date: 0001-01-01
summary: "HanLP Standard 分词器 #  hanlp_standard 分词器使用 HanLP 标准分词模式对中文文本进行分词，适合大多数中文搜索场景。
需要安装 analysis-hanlp 插件。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;hanlp_standard&#34; } } } } } 相关指南 #    HanLP 分词器  HanLP 标准分析器  文本分析基础  "
---


# HanLP Standard 分词器

`hanlp_standard` 分词器使用 HanLP 标准分词模式对中文文本进行分词，适合大多数中文搜索场景。

需要安装 `analysis-hanlp` 插件。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "hanlp_standard"
        }
      }
    }
  }
}
```

## 相关指南

- [HanLP 分词器]({{< relref "./hanlp" >}})
- [HanLP 标准分析器]({{< relref "../analyzers/hanlp-standard-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


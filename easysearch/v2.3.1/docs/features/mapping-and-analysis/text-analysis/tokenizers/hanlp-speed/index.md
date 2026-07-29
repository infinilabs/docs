---
title: "HanLP 极速分词器（HanLP Speed）"
date: 0001-01-01
summary: "HanLP Speed 分词器 #  hanlp_speed 分词器使用 HanLP 极速分词模式，牺牲一定精度换取更快的分词速度，适合对延迟敏感的在线搜索场景。
需要安装 analysis-hanlp 插件。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;hanlp_speed&#34; } } } } } 相关指南 #    HanLP 分词器  文本分析基础  "
---


# HanLP Speed 分词器

`hanlp_speed` 分词器使用 HanLP 极速分词模式，牺牲一定精度换取更快的分词速度，适合对延迟敏感的在线搜索场景。

需要安装 `analysis-hanlp` 插件。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "hanlp_speed"
        }
      }
    }
  }
}
```

## 相关指南

- [HanLP 分词器]({{< relref "./hanlp" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


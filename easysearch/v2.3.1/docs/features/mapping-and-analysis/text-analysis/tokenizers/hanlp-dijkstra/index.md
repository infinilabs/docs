---
title: "HanLP 最短路分词器（HanLP Dijkstra）"
date: 0001-01-01
summary: "HanLP Dijkstra 分词器 #  hanlp_dijkstra 分词器使用 HanLP Dijkstra 最短路径分词算法，基于词典的全切分方式。
需要安装 analysis-hanlp 插件。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;hanlp_dijkstra&#34; } } } } } 相关指南 #    HanLP 分词器  文本分析基础  "
---


# HanLP Dijkstra 分词器

`hanlp_dijkstra` 分词器使用 HanLP Dijkstra 最短路径分词算法，基于词典的全切分方式。

需要安装 `analysis-hanlp` 插件。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "hanlp_dijkstra"
        }
      }
    }
  }
}
```

## 相关指南

- [HanLP 分词器]({{< relref "./hanlp" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


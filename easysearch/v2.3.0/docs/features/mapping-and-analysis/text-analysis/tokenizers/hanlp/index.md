---
title: "HanLP 分词器概述"
date: 0001-01-01
summary: "HanLP 分词器 #  HanLP 是一个功能强大的中文自然语言处理库，通过 analysis-hanlp 插件 集成到 Easysearch 中。该插件提供了 7 种分词模式，覆盖从高速到高精度的各种需求。
前提条件 #  bin/easysearch-plugin install analysis-hanlp 分词模式一览 #     分词器名称 模式 速度 精度 适用场景      hanlp_standard 标准分词 ★★★ ★★★★ 通用中文分词    hanlp_index 索引分词 ★★★ ★★★ 索引时最大化召回    hanlp_nlp NLP 分词 ★★ ★★★★★ 命名实体识别    hanlp_crf CRF 分词 ★★ ★★★★★ 新词发现    hanlp_n_short N-最短路径 ★★ ★★★★ 歧义消解    hanlp_dijkstra 最短路径 ★★★ ★★★ 快速精确分词    hanlp_speed 极速分词 ★★★★★ ★★ 大数据量高吞吐    索引/搜索推荐搭配 #  PUT my-hanlp-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;hanlp_index_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;hanlp_index&#34; }, &#34;hanlp_search_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;hanlp_standard&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;hanlp_index_analyzer&#34;, &#34;search_analyzer&#34;: &#34;hanlp_search_analyzer&#34; } } } } 测试分词 #  GET /_analyze { &#34;tokenizer&#34;: &#34;hanlp_standard&#34;, &#34;text&#34;: &#34;中华人民共和国国歌&#34; } 模式选择建议 #     需求 推荐模式     通用场景 hanlp_standard   索引时最大召回 hanlp_index   人名/地名/机构名识别 hanlp_nlp   识别新词（训练语料外的词） hanlp_crf   追求最大吞吐量 hanlp_speed    相关链接 #    文本分析  "
---


# HanLP 分词器

HanLP 是一个功能强大的中文自然语言处理库，通过 analysis-hanlp 插件 集成到 Easysearch 中。该插件提供了 **7 种分词模式**，覆盖从高速到高精度的各种需求。

## 前提条件

```bash
bin/easysearch-plugin install analysis-hanlp
```

## 分词模式一览

| 分词器名称 | 模式 | 速度 | 精度 | 适用场景 |
|-----------|------|------|------|---------|
| [`hanlp_standard`]({{< relref "./hanlp-standard" >}}) | 标准分词 | ★★★ | ★★★★ | 通用中文分词 |
| [`hanlp_index`]({{< relref "./hanlp-index" >}}) | 索引分词 | ★★★ | ★★★ | 索引时最大化召回 |
| [`hanlp_nlp`]({{< relref "./hanlp-nlp" >}}) | NLP 分词 | ★★ | ★★★★★ | 命名实体识别 |
| [`hanlp_crf`]({{< relref "./hanlp-crf" >}}) | CRF 分词 | ★★ | ★★★★★ | 新词发现 |
| [`hanlp_n_short`]({{< relref "./hanlp-n-short" >}}) | N-最短路径 | ★★ | ★★★★ | 歧义消解 |
| [`hanlp_dijkstra`]({{< relref "./hanlp-dijkstra" >}}) | 最短路径 | ★★★ | ★★★ | 快速精确分词 |
| [`hanlp_speed`]({{< relref "./hanlp-speed" >}}) | 极速分词 | ★★★★★ | ★★ | 大数据量高吞吐 |

## 索引/搜索推荐搭配

```json
PUT my-hanlp-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "hanlp_index_analyzer": {
          "type": "custom",
          "tokenizer": "hanlp_index"
        },
        "hanlp_search_analyzer": {
          "type": "custom",
          "tokenizer": "hanlp_standard"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "hanlp_index_analyzer",
        "search_analyzer": "hanlp_search_analyzer"
      }
    }
  }
}
```

## 测试分词

```json
GET /_analyze
{
  "tokenizer": "hanlp_standard",
  "text": "中华人民共和国国歌"
}
```

## 模式选择建议

| 需求 | 推荐模式 |
|------|---------|
| 通用场景 | `hanlp_standard` |
| 索引时最大召回 | `hanlp_index` |
| 人名/地名/机构名识别 | `hanlp_nlp` |
| 识别新词（训练语料外的词） | `hanlp_crf` |
| 追求最大吞吐量 | `hanlp_speed` |

## 相关链接

- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


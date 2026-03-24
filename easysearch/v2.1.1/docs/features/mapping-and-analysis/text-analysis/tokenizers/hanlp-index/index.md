---
title: "HanLP Index 分词器"
date: 0001-01-01
summary: "HanLP Index 分词器 #  hanlp_index 分词器是 analysis-hanlp 插件 提供的索引模式分词器。它在标准分词的基础上对长词进行二次切分，生成更多子词项，适合索引时使用以提高召回率。
前提条件 #  需要安装 analysis-hanlp 插件：
bin/easysearch-plugin install analysis-hanlp 分词效果对比 #  以&quot;中华人民共和国国歌&quot;为例：
   分词器 输出词项     hanlp_standard 中华人民共和国、国歌   hanlp_index 中华、华人、人民、共和、共和国、中华人民共和国、国歌    hanlp_index 输出更细粒度的子词，确保无论用户搜索&quot;中华&quot;、&ldquo;人民&quot;还是&quot;共和国&quot;都能命中。
使用示例 #  在映射中指定 #  PUT my-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;hanlp_index_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;hanlp_index&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;hanlp_index_analyzer&#34;, &#34;search_analyzer&#34;: &#34;hanlp_standard&#34; } } } } 测试分词效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;hanlp_index&#34;, &#34;text&#34;: &#34;中华人民共和国国歌&#34; } 最佳实践 #     场景 推荐     索引时 使用 hanlp_index（最大化召回）   搜索时 使用 hanlp_standard 或 hanlp_nlp（精确匹配）    相关链接 #    HanLP Standard 分词器 — 标准分词模式  HanLP NLP 分词器 — 命名实体识别模式  HanLP 通用 — HanLP 分词器概述  文本分析  "
---


# HanLP Index 分词器

`hanlp_index` 分词器是 analysis-hanlp 插件 提供的索引模式分词器。它在标准分词的基础上对长词进行二次切分，生成更多子词项，适合**索引时**使用以提高召回率。

## 前提条件

需要安装 analysis-hanlp 插件：

```bash
bin/easysearch-plugin install analysis-hanlp
```

## 分词效果对比

以"中华人民共和国国歌"为例：

| 分词器 | 输出词项 |
|--------|---------|
| `hanlp_standard` | 中华人民共和国、国歌 |
| **`hanlp_index`** | **中华、华人、人民、共和、共和国、中华人民共和国、国歌** |

`hanlp_index` 输出更细粒度的子词，确保无论用户搜索"中华"、"人民"还是"共和国"都能命中。

## 使用示例

### 在映射中指定

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "hanlp_index_analyzer": {
          "type": "custom",
          "tokenizer": "hanlp_index"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "hanlp_index_analyzer",
        "search_analyzer": "hanlp_standard"
      }
    }
  }
}
```

### 测试分词效果

```json
GET /_analyze
{
  "tokenizer": "hanlp_index",
  "text": "中华人民共和国国歌"
}
```

## 最佳实践

| 场景 | 推荐 |
|------|------|
| **索引时** | 使用 `hanlp_index`（最大化召回） |
| **搜索时** | 使用 `hanlp_standard` 或 `hanlp_nlp`（精确匹配） |

## 相关链接

- [HanLP Standard 分词器]({{< relref "./hanlp-standard" >}}) — 标准分词模式
- [HanLP NLP 分词器]({{< relref "./hanlp-nlp" >}}) — 命名实体识别模式
- [HanLP 通用]({{< relref "./hanlp" >}}) — HanLP 分词器概述
- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


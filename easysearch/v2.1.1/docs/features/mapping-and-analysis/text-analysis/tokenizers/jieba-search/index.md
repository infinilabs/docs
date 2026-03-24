---
title: "Jieba Search 分词器"
date: 0001-01-01
summary: "Jieba Search 分词器 #  jieba_search 分词器是 analysis-jieba 插件 提供的搜索模式分词器。它在精确分词的基础上对长词不做额外切分，适合搜索时使用。
前提条件 #  bin/easysearch-plugin install analysis-jieba 与 jieba_index 的对比 #     分词器 模式 以&quot;中华人民共和国&quot;为例 适用场景     jieba_search 精确模式 中华人民共和国 搜索时   jieba_index 索引模式 中华、华人、人民、共和、共和国、中华人民共和国 索引时    使用示例 #  索引/搜索搭配 #  PUT my-jieba-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;jieba_idx&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;jieba_index&#34; }, &#34;jieba_srch&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;jieba_search&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;jieba_idx&#34;, &#34;search_analyzer&#34;: &#34;jieba_srch&#34; } } } } 测试分词 #  GET /_analyze { &#34;tokenizer&#34;: &#34;jieba_search&#34;, &#34;text&#34;: &#34;中华人民共和国国歌&#34; } 相关链接 #    Jieba Index 分词器 — 索引模式  文本分析  "
---


# Jieba Search 分词器

`jieba_search` 分词器是 analysis-jieba 插件 提供的**搜索模式**分词器。它在精确分词的基础上对长词不做额外切分，适合**搜索时**使用。

## 前提条件

```bash
bin/easysearch-plugin install analysis-jieba
```

## 与 jieba_index 的对比

| 分词器 | 模式 | 以"中华人民共和国"为例 | 适用场景 |
|--------|------|---------------------|---------|
| **`jieba_search`** | 精确模式 | 中华人民共和国 | 搜索时 |
| `jieba_index` | 索引模式 | 中华、华人、人民、共和、共和国、中华人民共和国 | 索引时 |

## 使用示例

### 索引/搜索搭配

```json
PUT my-jieba-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "jieba_idx": {
          "type": "custom",
          "tokenizer": "jieba_index"
        },
        "jieba_srch": {
          "type": "custom",
          "tokenizer": "jieba_search"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "jieba_idx",
        "search_analyzer": "jieba_srch"
      }
    }
  }
}
```

### 测试分词

```json
GET /_analyze
{
  "tokenizer": "jieba_search",
  "text": "中华人民共和国国歌"
}
```

## 相关链接

- [Jieba Index 分词器]({{< relref "./jieba-index" >}}) — 索引模式
- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


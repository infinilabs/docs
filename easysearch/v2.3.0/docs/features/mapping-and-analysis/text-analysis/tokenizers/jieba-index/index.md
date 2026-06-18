---
title: "Jieba Index 分词器"
date: 0001-01-01
summary: "Jieba Index 分词器 #  jieba_index 分词器是 analysis-jieba 插件 提供的索引模式分词器。它在精确分词的基础上对长词进行二次切分，生成更多子词项，适合索引时使用以最大化召回率。
前提条件 #  bin/easysearch-plugin install analysis-jieba 分词效果 #  以&quot;中华人民共和国国歌&quot;为例：
   分词器 输出     jieba_search 中华人民共和国、国歌   jieba_index 中华、华人、人民、共和、共和国、中华人民共和国、国歌    使用示例 #  基本用法 #  PUT my-jieba-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;jieba_idx&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;jieba_index&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;jieba_idx&#34;, &#34;search_analyzer&#34;: &#34;jieba_srch&#34; } } } } 测试分词 #  GET /_analyze { &#34;tokenizer&#34;: &#34;jieba_index&#34;, &#34;text&#34;: &#34;中华人民共和国国歌&#34; } 最佳实践 #     场景 分词器     索引时 jieba_index（最大化召回）   搜索时 jieba_search（精确匹配）    相关链接 #    Jieba Search 分词器 — 搜索模式  文本分析  "
---


# Jieba Index 分词器

`jieba_index` 分词器是 analysis-jieba 插件 提供的**索引模式**分词器。它在精确分词的基础上对长词进行二次切分，生成更多子词项，适合**索引时**使用以最大化召回率。

## 前提条件

```bash
bin/easysearch-plugin install analysis-jieba
```

## 分词效果

以"中华人民共和国国歌"为例：

| 分词器 | 输出 |
|--------|------|
| `jieba_search` | 中华人民共和国、国歌 |
| **`jieba_index`** | **中华、华人、人民、共和、共和国、中华人民共和国、国歌** |

## 使用示例

### 基本用法

```json
PUT my-jieba-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "jieba_idx": {
          "type": "custom",
          "tokenizer": "jieba_index"
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
  "tokenizer": "jieba_index",
  "text": "中华人民共和国国歌"
}
```

## 最佳实践

| 场景 | 分词器 |
|------|--------|
| **索引时** | `jieba_index`（最大化召回） |
| **搜索时** | `jieba_search`（精确匹配） |

## 相关链接

- [Jieba Search 分词器]({{< relref "./jieba-search" >}}) — 搜索模式
- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


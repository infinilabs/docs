---
title: "简繁转换字符过滤器（ST Convert）"
date: 0001-01-01
summary: "ST Convert 字符过滤器 #  stconvert 字符过滤器在分词之前对中文文本进行简体与繁体之间的转换，使简繁体内容在搜索时可以互相匹配。
 需要插件：此过滤器由 analysis-stconvert 插件提供，Easysearch 默认已集成。
 转换方向 #     convert_type 说明     s2t 默认值。简体 → 繁体   t2s 繁体 → 简体    使用示例 #  简体转繁体（默认） #  PUT /my-chinese-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;s2t_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;char_filter&#34;: [&#34;stconvert&#34;] } } } } } 繁体转简体 #  PUT /my-chinese-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;char_filter&#34;: { &#34;t2s_filter&#34;: { &#34;type&#34;: &#34;stconvert&#34;, &#34;convert_type&#34;: &#34;t2s&#34; } }, &#34;analyzer&#34;: { &#34;t2s_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;char_filter&#34;: [&#34;t2s_filter&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;keyword&#34;, &#34;char_filter&#34;: [ { &#34;type&#34;: &#34;stconvert&#34;, &#34;convert_type&#34;: &#34;t2s&#34; } ], &#34;text&#34;: &#34;國際化標準&#34; } 结果：國際化標準 → 国际化标准"
---


# ST Convert 字符过滤器

`stconvert` 字符过滤器在**分词之前**对中文文本进行简体与繁体之间的转换，使简繁体内容在搜索时可以互相匹配。

> **需要插件**：此过滤器由 `analysis-stconvert` 插件提供，Easysearch 默认已集成。

## 转换方向

| `convert_type` | 说明 |
|----------------|------|
| `s2t` | **默认值**。简体 → 繁体 |
| `t2s` | 繁体 → 简体 |

## 使用示例

### 简体转繁体（默认）

```json
PUT /my-chinese-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "s2t_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": ["stconvert"]
        }
      }
    }
  }
}
```

### 繁体转简体

```json
PUT /my-chinese-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "t2s_filter": {
          "type": "stconvert",
          "convert_type": "t2s"
        }
      },
      "analyzer": {
        "t2s_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": ["t2s_filter"]
        }
      }
    }
  }
}
```

### 测试效果

```json
GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    {
      "type": "stconvert",
      "convert_type": "t2s"
    }
  ],
  "text": "國際化標準"
}
```

结果：`國際化標準` → `国际化标准`

## 参数说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `convert_type` | String | `s2t` | 转换方向：`s2t`（简→繁）或 `t2s`（繁→简） |

## 字符过滤器 vs 词元过滤器

`analysis-stconvert` 插件同时提供了字符过滤器、词元过滤器和分词器三种形式：

| 形式 | 类型名 | 处理阶段 | 说明 |
|------|--------|---------|------|
| **字符过滤器** | `stconvert` | 分词前 | 先转换再分词，分词结果更准确 |
| 词元过滤器 | `stconvert` | 分词后 | 分词后再转换 |
| 分词器 | `stconvert` | 分词阶段 | 边分词边转换 |

> **建议**：中文场景推荐使用字符过滤器版本，先完成简繁转换再交给 IK 等分词器处理，可以避免因简繁差异导致的分词不一致。

## 典型应用场景

### 简繁体统一索引

将简繁体统一为简体索引，用户用任一种写法都能搜到：

```json
PUT /chinese-unified
{
  "settings": {
    "analysis": {
      "char_filter": {
        "t2s": {
          "type": "stconvert",
          "convert_type": "t2s"
        }
      },
      "analyzer": {
        "unified_chinese": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": ["t2s"],
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "unified_chinese"
      }
    }
  }
}
```

这样索引 `國際化` 和 `国际化` 都会被归一化为 `国际化`，实现简繁体互搜。


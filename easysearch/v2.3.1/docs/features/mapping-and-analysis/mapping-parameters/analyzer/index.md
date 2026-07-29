---
title: "分析器参数（Analyzer）"
date: 0001-01-01
summary: "Analyzer 参数 #  analyzer 映射参数用于定义在索引和搜索期间应用于文本字段的文本分析过程。
相关指南（先读这些） #    映射基础  文本分析基础  代码样例 #  以下示例配置定义了一个名为 my_custom_analyzer 的自定义分词器：
PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_custom_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_stop_filter&#34;, &#34;my_stemmer&#34; ] } }, &#34;filter&#34;: { &#34;my_stop_filter&#34;: { &#34;type&#34;: &#34;stop&#34;, &#34;stopwords&#34;: [&#34;the&#34;, &#34;a&#34;, &#34;and&#34;, &#34;or&#34;] }, &#34;my_stemmer&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;language&#34;: &#34;english&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_text_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_custom_analyzer&#34;, &#34;search_analyzer&#34;: &#34;standard&#34;, &#34;search_quote_analyzer&#34;: &#34;my_custom_analyzer&#34; } } } } 在此示例中，my_custom_analyzer 使用标准分词器，将所有标记转换为小写，应用自定义停用词过滤器，并应用英语词干提取器。"
---


# Analyzer 参数

`analyzer` 映射参数用于定义在索引和搜索期间应用于文本字段的文本分析过程。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 代码样例

以下示例配置定义了一个名为 `my_custom_analyzer` 的自定义分词器：

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_stop_filter",
            "my_stemmer"
          ]
        }
      },
      "filter": {
        "my_stop_filter": {
          "type": "stop",
          "stopwords": ["the", "a", "and", "or"]
        },
        "my_stemmer": {
          "type": "stemmer",
          "language": "english"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_text_field": {
        "type": "text",
        "analyzer": "my_custom_analyzer",
        "search_analyzer": "standard",
        "search_quote_analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

在此示例中，`my_custom_analyzer` 使用标准分词器，将所有标记转换为小写，应用自定义停用词过滤器，并应用英语词干提取器。

您可以映射一个文本字段，使其在索引和搜索操作中都使用此自定义分词器：

```
"mappings": {
  "properties": {
    "my_text_field": {
      "type": "text",
      "analyzer": "my_custom_analyzer"
    }
  }
}
```

## 三种 Analyzer 参数

Easysearch 支持在映射中指定三种不同的分析器，作用于不同阶段：

| 参数                     | 作用阶段             | 说明                                                         |
| ------------------------ | -------------------- | ------------------------------------------------------------ |
| `analyzer`               | 索引时 + 搜索时      | 同时用于索引和搜索的默认分析器。                              |
| `search_analyzer`        | 仅搜索时             | 覆盖搜索时使用的分析器。适合索引和搜索使用不同分析策略的场景。 |
| `search_quote_analyzer`  | 短语搜索时           | 用于 `match_phrase` 等短语查询。适合短语搜索不做词干提取。     |

### 示例：索引和搜索使用不同分析器

一个常见场景是索引时使用同义词扩展，但搜索时使用标准分析器：

```
PUT products
{
  "settings": {
    "analysis": {
      "analyzer": {
        "synonym_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "my_synonyms"]
        }
      },
      "filter": {
        "my_synonyms": {
          "type": "synonym",
          "synonyms": ["手机,手持电话,移动电话"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "product_name": {
        "type": "text",
        "analyzer": "synonym_analyzer",
        "search_analyzer": "standard"
      }
    }
  }
}
```

## 常用内置分析器

| 分析器名称   | 描述                                                    | 适用场景            |
| ------------ | ------------------------------------------------------- | ------------------- |
| `standard`   | 按 Unicode 文本分割规则分词，转小写，去停用词（可选）   | 大多数西语文本      |
| `simple`      | 按非字母字符分割，转小写                                | 简单文本            |
| `whitespace` | 按空格分割，不做其他处理                                | 结构化文本          |
| `keyword`    | 不分词，整个字段值作为一个词项                          | 精确值              |
| `ik_smart`   | IK 智能分词（中文）                                      | 中文搜索（推荐）    |
| `ik_max_word`| IK 最大化分词（中文）                                    | 中文索引（高召回）  |

> **提示**：对于中文内容，推荐使用 IK 分词器。可以使用 `_analyze` API 测试分析效果：
>
> ```
> POST _analyze
> {
>   "analyzer": "ik_smart",
>   "text": "中国人民银行"
> }
> ```


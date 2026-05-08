---
title: "More Like This 查询"
date: 0001-01-01
summary: "More Like This 查询 #  使用 more_like_this 查询查找与一个或多个给定文档相似的文档。这对于推荐引擎、内容发现以及识别数据集中的相关项目很有用。
more_like_this 查询分析输入文档或文本，并选择最能描述它们的词项。然后，它搜索包含这些重要词项的其他文档。
相关指南（先读这些） #    相关性与打分策略  Query DSL 基础  前提条件 #  在使用 more_like_this 查询之前，请确保您目标字段已索引，且其数据类型为 text 或 keyword 。
如果您在 like 部分引用文档，Easysearch 需要访问其内容。这通常通过 _source 字段完成，该字段默认启用。如果 _source 被禁用，您必须单独存储这些字段，或配置它们以保存 term_vector 数据。
 在索引文档时保存 term_vector 信息可以大大加速 more_like_this 查询，因为引擎可以直接检索重要词项，而无需在查询时重新分析字段文本。
 示例：无词向量优化 #  使用以下映射创建名为 articles-basic 的索引：
PUT /articles-basic { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34; } } } } 添加示例文档："
---


# More Like This 查询

使用 `more_like_this` 查询查找与一个或多个给定文档相似的文档。这对于推荐引擎、内容发现以及识别数据集中的相关项目很有用。

`more_like_this` 查询分析输入文档或文本，并选择最能描述它们的词项。然后，它搜索包含这些重要词项的其他文档。

## 相关指南（先读这些）

- [相关性与打分策略]({{< relref "/docs/features/fulltext-search/relevance/_index.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 前提条件

在使用 `more_like_this` 查询之前，请确保您目标字段已索引，且其数据类型为 `text` 或 `keyword` 。

如果您在 `like` 部分引用文档，Easysearch 需要访问其内容。这通常通过 `_source` 字段完成，该字段默认启用。如果 `_source` 被禁用，您必须单独存储这些字段，或配置它们以保存 `term_vector` 数据。

> 在索引文档时保存 `term_vector` 信息可以大大加速 `more_like_this` 查询，因为引擎可以直接检索重要词项，而无需在查询时重新分析字段文本。

## 示例：无词向量优化

使用以下映射创建名为 `articles-basic` 的索引：

```
PUT /articles-basic
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" }
    }
  }
}
```

添加示例文档：

```
POST /articles-basic/_bulk
{ "index": { "_id": 1 }}
{ "title": "Exploring the Sahara Desert", "content": "Sand dunes and vast landscapes." }
{ "index": { "_id": 2 }}
{ "title": "Amazon Rainforest Tour", "content": "Dense jungle and exotic wildlife." }
{ "index": { "_id": 3 }}
{ "title": "Mountain Adventures", "content": "Snowy peaks and hiking trails." }
```

使用以下请求进行查询：

```
GET /articles-basic/_search
{
  "query": {
    "more_like_this": {
      "fields": ["content"],
      "like": "jungle wildlife",
      "min_term_freq": 1,
      "min_doc_freq": 1
    }
  }
}
```

`more_like_this` 查询在 `content` 字段中搜索 `jungle` 和 `wildlife` ，仅匹配一个文档：

```
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.9616582,
    "hits": [
      {
        "_index": "articles-basic",
        "_id": "2",
        "_score": 1.9616582,
        "_source": {
          "title": "Amazon Rainforest Tour",
          "content": "Dense jungle and exotic wildlife."
        }
      }
    ]
  }
}
```

## 示例：词项向量优化

使用以下映射创建名为 `articles-optimized` 的索引：

```
PUT /articles-optimized
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "term_vector": "with_positions_offsets"
      },
      "content": {
        "type": "text",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}
```

将样本文档插入优化索引：

```
POST /articles-optimized/_bulk
{ "index": { "_id": "a1" } }
{ "name": "Diana", "alias": "Wonder Woman", "quote": "Justice will come when it is deserved." }
{ "index": { "_id": "a2" } }
{ "name": "Clark", "alias": "Superman", "quote": "Even in the darkest times, hope cuts through." }
{ "index": { "_id": "a3" } }
{ "name": "Bruce", "alias": "Batman", "quote": "I am vengeance. I am the night. I am Batman!" }
```

查找 quote 字段中包含与“dark”和“night”相似的词的文档：

```
GET /articles-optimized/_search
{
  "query": {
    "more_like_this": {
      "fields": ["quote"],
      "like": "dark night",
      "min_term_freq": 1,
      "min_doc_freq": 1
    }
  }
}
```

`more_like_this` 查询搜索 dark 和 night 并返回以下命中结果：

```
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.2363393,
    "hits": [
      {
        "_index": "articles-optimized",
        "_id": "a3",
        "_score": 1.2363393,
        "_source": {
          "name": "Bruce",
          "alias": "Batman",
          "quote": "I am vengeance. I am the night. I am Batman!"
        }
      }
    ]
  }
}

```

## 示例：使用多个文档和文本输入

`more_like_this` 查询允许你在 `like` 参数中提供多个来源。你可以将自由文本与索引中的文档结合使用。如果你希望搜索结合多个示例的相关性信号，这会很有用。

在以下示例中，直接提供了一个自定义文档。此外，还包括了来自 `heroes` 索引的具有 ID 5 的现有文档：

```
GET /articles-optimized/_search
{
  "query": {
    "more_like_this": {
      "fields": ["name", "alias"],
      "like": [
        {
          "doc": {
            "name": "Diana",
            "alias": "Wonder Woman",
            "quote": "Courage is not the absence of fear, but the triumph over it."
          }
        },
        {
          "_index": "heroes",
          "_id": "5"
        }
      ],
      "min_term_freq": 1,
      "min_doc_freq": 1,
      "max_query_terms": 25
    }
  }
}
```

返回的结果包含与查询中提供的 `name` 和 `alias` 字段最相似的文章：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 2.140194,
    "hits": [
      {
        "_index": "articles-optimized",
        "_id": "a1",
        "_score": 2.140194,
        "_source": {
          "name": "Diana",
          "alias": "Wonder Woman",
          "quote": "Justice will come when it is deserved."
        }
      }
    ]
  }
}

```

> 当你希望根据一个尚未完全索引的新概念来提升结果，同时还想将其与现有索引文档中的知识结合起来时，请使用此模式。

## 参数说明

`more_like_this` 查询的唯一必需参数是 `like` 。其余参数具有默认值，但允许精细调整。以下为主要参数类别。

### 文档输入参数

下表指定了文档输入参数。

| 参数     | 必需/可选 | 数据类型           | 描述                                                                                                                           |
| -------- | --------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| `like`   | 必需      | 字符串或对象的数组 | 定义要查找相似文档的文本或文档。您可以输入自由文本、索引中的真实文档或人工文档。除非被覆盖，否则与字段关联的分词器会处理文本。 |
| `unlike` | 可选      | 字符串或对象的数组 | 提供文本或文档，其中的词项应被排除，不用于影响查询。可用于指定负例。                                                           |
| `fields` | 可选      | 字符串数组         | 列出用于分析文本的字段。如果未指定，则使用所有字段。                                                                           |

### 词项选择参数

| 参数              | 必需/可选 | 数据类型   | 描述                                                                                           |
| ----------------- | --------- | ---------- | ---------------------------------------------------------------------------------------------- |
| `max_query_terms` | 可选      | Integer    | 设置从输入中选择的最大词数。值越高，精确度越高，但执行速度会变慢。默认值为 25 。               |
| `min_term_freq`   | 可选      | Integer    | 在输入中出现的次数少于此值的词将被忽略。默认值为 2 。                                          |
| `min_doc_freq`    | 可选      | Integer    | 在少于此值数量的文档中出现的词将被忽略。默认值为 5 。                                          |
| `max_doc_freq`    | 可选      | Integer    | 在超过此限制数量的文档中出现的词将被忽略。可用于避免非常常见的词。默认值为无限制（2^31 - 1）。 |
| `min_word_length` | 可选      | Integer    | 忽略比这个值更短的词。默认值是 0 。                                                            |
| `max_word_length` | 可选      | Integer    | 忽略比这个值更长的词。默认值是无限制。                                                         |
| `stop_words`      | 可选      | 字符串数组 | 定义在选取词项时完全忽略的一系列单词。                                                         |
| `analyzer`        | 可选      | String     | 用于处理输入文本的自定义分词器。默认为 fields 中列出的第一个字段的分词器。                     |

### 查询形成参数

| 参数                        | 必需/可选 | 数据类型 | 描述                                                                                                                             |
| --------------------------- | --------- | -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `minimum_should_match`      | 可选      | String   | 指定最终查询中必须匹配的最小词数。值可以是百分比或固定数字。有助于微调召回率和精确率之间的平衡。默认值是 30%                     |
| `fail_on_unsupported_field` | 可选      | Boolean  | 确定如果目标字段中有一个不是兼容类型（ text 或 keyword ），是否抛出错误。设置为 false 以静默方式跳过不支持的字段。默认是 true 。 |
| `boost_terms`               | 可选      | Float    | 根据所选词项的词频-逆文档频率（TF–IDF）权重对词项应用提升。任何大于 0 的值都会使用指定的因子激活词项提升。默认是 0 。            |
| `include`                   | 可选      | Boolean  | 如果 true ，则 like 中提供的源文档包含在结果命中中。默认是 false 。                                                              |
| `boost	`                     | 可选      | Float    | 乘以整个 more_like_this 查询的相关性分数。默认值为 1.0 。                                                                        |


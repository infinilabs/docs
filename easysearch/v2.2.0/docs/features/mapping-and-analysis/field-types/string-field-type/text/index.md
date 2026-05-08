---
title: "文本字段类型（Text）"
date: 0001-01-01
summary: "Text 字段类型 #  text 字段类型包含经过分析器分析的字符串。它用于全文搜索，因为它允许部分匹配。对多个词条的搜索可以匹配其中的一部分而不是全部。根据分析器的不同，搜索结果可以是大小写不敏感的、词干化的、去除停用词的、应用同义词的等等。
 提示：如果您需要进行精确值搜索，请将字段映射为 keyword 类型。
 相关指南（先读这些） #    映射基础  映射模式  全文搜索  代码样例 #  创建一个带有 text 字段的映射：
PUT movies { &#34;mappings&#34; : { &#34;properties&#34; : { &#34;title&#34; : { &#34;type&#34; : &#34;text&#34; } } } } 参数说明 #  下表列出了 text 字段类型接受的参数。所有参数都是可选的。
   参数 描述     analyzer 用于此字段的分词器。默认情况下，它将在索引时和搜索时使用。要在搜索时覆盖它，请设置 search_analyzer 参数。默认是 standard 分词器，它使用基于语法的分词，并基于 Unicode 文本分段算法。   boost 浮点值，指定此字段对相关性分数的权重。大于 1."
---


# Text 字段类型

`text` 字段类型包含经过分析器分析的字符串。它用于全文搜索，因为它允许部分匹配。对多个词条的搜索可以匹配其中的一部分而不是全部。根据分析器的不同，搜索结果可以是大小写不敏感的、词干化的、去除停用词的、应用同义词的等等。

> **提示**：如果您需要进行精确值搜索，请将字段映射为 [keyword](./keyword.md) 类型。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

## 代码样例

创建一个带有 text 字段的映射：

```json
PUT movies
{
  "mappings" : {
    "properties" : {
      "title" : {
        "type" : "text"
      }
    }
  }
}
```

## 参数说明

下表列出了 text 字段类型接受的参数。所有参数都是可选的。

| 参数                       | 描述                                                                                                                                                                                                                                                                                                                                    |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| analyzer                   | 用于此字段的分词器。默认情况下，它将在索引时和搜索时使用。要在搜索时覆盖它，请设置 `search_analyzer` 参数。默认是 `standard` 分词器，它使用基于语法的分词，并基于 Unicode 文本分段算法。                                                                                                                                                |
| boost                      | 浮点值，指定此字段对相关性分数的权重。大于 1.0 的值会增加字段的相关性。0.0 到 1.0 之间的值会降低字段的相关性。默认值为 1.0。                                                                                                                                                                                                            |
| eager_global_ordinals      | 指定是否应在刷新时立即加载全局序号。如果该字段经常用于聚合，此参数应设置为 `true`。默认值为 `false`。                                                                                                                                                                                                                                   |
| fielddata                  | 布尔值，指定是否访问此字段的已分析标记以进行排序、聚合和脚本编写。默认值为 `false`。                                                                                                                                                                                                                                                    |
| fielddata_frequency_filter | JSON 对象，指定仅将文档频率在 `min` 和 `max` 值之间的已分析标记加载到内存中（以绝对数字或百分比提供）。频率按段计算。参数：`min`、`max`、`min_segment_size`。默认加载所有已分析的标记。                                                                                                                                                 |
| fields                     | 要以多种方式索引同一个字符串（例如，作为 keyword 和 text），请提供 fields 参数。您可以指定字段的一个版本用于搜索，另一个版本用于排序和聚合。                                                                                                                                                                                            |
| index                      | 布尔值，指定字段是否应可搜索。默认值为 `true`。                                                                                                                                                                                                                                                                                         |
| index_options              | 指定要存储在索引中用于搜索和突出显示的信息。有效值：docs（仅文档编号）、freqs（文档编号和词频）、positions（文档编号、词频和词位置）、offsets（文档编号、词频、词位置以及开始和结束字符偏移量）。默认值为 positions。                                                                                                                   |
| index_phrases              | 布尔值，指定是否单独索引 2-gram。2-gram 是此字段字符串中两个连续单词的组合。导致精确短语查询更快但索引更大。当不删除停用词时效果最好。默认值为 `false`。                                                                                                                                                                                |
| index_prefixes             | JSON 对象，指定单独索引词条前缀。前缀中的字符数在 `min_chars` 和 `max_chars` 之间（包含）。导致前缀搜索更快但索引更大。可选参数：`min_chars`、`max_chars`。默认 `min_chars` 为 2，`max_chars` 为 5。                                                                                                                                    |
| meta                       | 接受此字段的元数据。                                                                                                                                                                                                                                                                                                                    |
| norms                      | 布尔值，指定在计算相关性分数时是否应使用字段长度。默认值为 `true`。norms 对全文搜索的评分非常重要，但会占用额外的磁盘空间。如果不需要对该字段进行评分（例如仅用于过滤），可设为 `false`。                                                                                                                                                                                                                    |
| position_increment_gap     | 当文本字段被分析时，它们被分配位置。如果一个字段包含一个字符串数组，并且这些位置是连续的，这将导致可能跨不同数组元素匹配。为防止这种情况，在连续的数组元素之间插入一个人工间隙。您可以通过指定整数 `position_increment_gap` 来更改此间隙。注意：如果 `slop` 大于 `position_element_gap`，可能会发生跨不同数组元素的匹配。默认值为 100。 |
| similarity                 | 用于计算相关性分数的排名算法。默认值为 `BM25`。                                                                                                                                                                                                                                                                                         |
| term_vector                | 布尔值，指定是否应存储此字段的词向量。默认值为 `no`。                                                                                                                                                                                                                                                                                   |

## 词向量参数

词向量在分析过程中产生。它包含：

- 词条列表
- 每个词条的序数位置
- 搜索字符串在字段中的起始和结束字符偏移量
- 负载 Payloads（如果可用）。每个词条可以有与词条位置相关的自定义二进制数据

`term_vector` 字段包含一个接受以下参数的 JSON 对象：

| 参数                            | 存储的值                       |
| ------------------------------- | ------------------------------ |
| no                              | 无。这是默认值。               |
| yes                             | 字段中的词条。                 |
| with_offsets                    | 词条和字符偏移量。             |
| with_positions_offsets          | 词条、位置和字符偏移量。       |
| with_positions_offsets_payloads | 词条、位置、字符偏移量和负载。 |
| with_positions                  | 词条和位置。                   |
| with_positions_payloads         | 词条、位置和负载。             |

> 存储位置对于邻近查询很有用。存储字符偏移量对于突出显示很有用。

### 词向量参数示例

创建一个带有存储字符偏移量的词向量的文本字段的映射：

```json
PUT testindex
{
  "mappings" : {
    "properties" : {
      "dob" : {
        "type" : "text",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}
```

索引一个带有文本字段的文档：

```json
PUT testindex/_doc/1
{
  "dob" : "The patient's date of birth."
}
```

查询"date of birth"并在原始字段中突出显示它：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "dob": "date of birth"
    }
  },
  "highlight": {
    "fields": {
      "dob": {}
    }
  }
}
```

返回内容中的"date of birth"被突出显示：

```json
{
  "took": 854,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.8630463,
    "hits": [
      {
        "_index": "testindex",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.8630463,
        "_source": {
          "text": "The patient's date of birth."
        },
        "highlight": {
          "text": ["The patient's <em>date</em> <em>of</em> <em>birth</em>."]
        }
      }
    ]
  }
}
```


---
title: "Intervals 查询"
date: 0001-01-01
summary: "Intervals 查询 #  intervals 查询根据匹配词的邻近度和顺序来匹配文档。它将一组匹配规则应用于指定字段中的词。该查询生成跨越文本中词的最小间隔序列。你可以组合间隔并按父源进行过滤。
相关指南（先读这些） #    邻近匹配  全文搜索  考虑一个包含以下文档的索引：
PUT testindex/_doc/1 { &#34;title&#34;: &#34;key-value pairs are efficiently stored in a hash table&#34; } PUT /testindex/_doc/2 { &#34;title&#34;: &#34;store key-value pairs in a hash map&#34; } 例如，以下查询搜索包含短语 key-value pairs （词之间没有间隔）后跟 hash table 或 hash map 的文档：
GET /testindex/_search { &#34;query&#34;: { &#34;intervals&#34;: { &#34;title&#34;: { &#34;all_of&#34;: { &#34;ordered&#34;: true, &#34;intervals&#34;: [ { &#34;match&#34;: { &#34;query&#34;: &#34;key-value pairs&#34;, &#34;max_gaps&#34;: 0, &#34;ordered&#34;: true } }, { &#34;any_of&#34;: { &#34;intervals&#34;: [ { &#34;match&#34;: { &#34;query&#34;: &#34;hash table&#34; } }, { &#34;match&#34;: { &#34;query&#34;: &#34;hash map&#34; } } ] } } ] } } } } } 该查询返回两个文档："
---


# Intervals 查询

`intervals` 查询根据匹配词的邻近度和顺序来匹配文档。它将一组匹配规则应用于指定字段中的词。该查询生成跨越文本中词的最小间隔序列。你可以组合间隔并按父源进行过滤。

## 相关指南（先读这些）

- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

考虑一个包含以下文档的索引：

```
PUT testindex/_doc/1
{
  "title": "key-value pairs are efficiently stored in a hash table"
}

PUT /testindex/_doc/2
{
  "title": "store key-value pairs in a hash map"
}
```

例如，以下查询搜索包含短语 `key-value pairs` （词之间没有间隔）后跟 `hash table` 或 `hash map` 的文档：

```
GET /testindex/_search
{
  "query": {
    "intervals": {
      "title": {
        "all_of": {
          "ordered": true,
          "intervals": [
            {
              "match": {
                "query": "key-value pairs",
                "max_gaps": 0,
                "ordered": true
              }
            },
            {
              "any_of": {
                "intervals": [
                  {
                    "match": {
                      "query": "hash table"
                    }
                  },
                  {
                    "match": {
                      "query": "hash map"
                    }
                  }
                ]
              }
            }
          ]
        }
      }
    }
  }
}

```

该查询返回两个文档：

```
{
  "took": 1011,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.25,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.25,
        "_source": {
          "title": "store key-value pairs in a hash map"
        }
      },
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.14285713,
        "_source": {
          "title": "key-value pairs are efficiently stored in a hash table"
        }
      }
    ]
  }
}
```

## 参数说明

该查询将字段名称（ `<field>` ）作为顶级参数接受：

```
GET _search
{
  "query": {
    "intervals": {
      "<field>": {
        ...
      }
    }
  }
}
```

`<field>` 接受以下规则对象，这些规则对象用于根据词项、顺序和邻近度来匹配文档。

| 规则       | 描述                                         |
| ---------- | -------------------------------------------- |
| `match`    | 匹配分析文本。                               |
| `prefix`   | 匹配以指定字符集开头的词项。                 |
| `wildcard` | 使用通配符模式匹配词项。                     |
| `regexp`   | 使用正则表达式模式匹配词项。                 |
| `fuzzy`    | 在指定的编辑距离内匹配与提供词项相似的词项。 |
| `all_of`   | 使用合取（ `AND` ）组合多个规则。            |
| `any_of`   | 使用析取（ `OR` ）组合多个规则。             |

### match 规则

`match` 规则匹配分析后的文本。下表列出了 `match` 规则支持的所有参数。

| 参数        | 必需/可选 | 数据类型           | 描述                                                                                                                                                                                                                                                           |
| ----------- | --------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`     | 必需      | String             | 要搜索的文本。                                                                                                                                                                                                                                                 |
| `analyzer`  | 可选      | String             | 用于分析 query 文本的解析器。默认为为 <field> 指定的解析器。                                                                                                                                                                                                   |
| `filter`    | 可选      | 间隔过滤器规则对象 | 用于过滤返回间隔的规则。                                                                                                                                                                                                                                       |
| `max_gaps`  | 可选      | Integer            | 匹配词项之间允许的最大位置数。距离超过 max_gaps 的词项不被视为匹配。如果 max_gaps 未指定或设置为 -1 ，无论位置如何，词项都被视为匹配。如果 max_gaps 设置为 0 ，匹配的词项必须相邻出现。默认为 -1 。                                                            |
| `ordered`   | 可选      | Boolean            | 指定匹配的词项是否必须按指定顺序出现。默认值为 false 。                                                                                                                                                                                                        |
| `use_field` | 可选      | String             | 指定在此字段中搜索，而不是在顶层字段中搜索。词项使用为此字段指定的搜索分词器进行分析。通过指定 `use_field`，您可以像它们是同一个字段一样跨多个字段进行搜索。例如，如果您将相同的文本索引到词干化和未词干化的字段中，您可以搜索与未词干化词项相邻的词干化词项。 |

### prefix 规则

`prefix` 规则匹配以指定字符集（前缀）开头的词项。前缀可以扩展以匹配最多 128 个词项。如果前缀匹配的词项超过 128 个，则返回错误。下表列出了 `prefix` 规则支持的所有参数。

| 参数        | 必需/可选 | 数据类型 | 描述                                                                                                 |
| ----------- | --------- | -------- | ---------------------------------------------------------------------------------------------------- |
| `prefix`    | 必需      | String   | 用于匹配词的前缀。                                                                                   |
| `analyzer`  | 可选      | String   | 用于分析 prefix 文本的解析器。默认为为 <field> 指定的解析器。                                        |
| `use_field` | 可选      | String   | 指定搜索此字段而不是顶层字段。`prefix`使用此字段指定的搜索分词器进行规范化，除非您指定了`analyzer`。 |

### wildcard 规则

`wildcard` 规则使用通配符模式匹配词项。通配符模式最多可以扩展匹配 128 个词项。如果模式匹配的词项超过 128 个，将返回错误。下表列出了 `wildcard` 规则支持的所有参数。

| 参数        | 必需/可选 | 数据类型 | 描述                                                                                                  |
| ----------- | --------- | -------- | ----------------------------------------------------------------------------------------------------- |
| `pattern`   | 必需      | String   | 用于匹配词项的通配符模式。指定 `?` 匹配任意单个字符，或指定 `*` 匹配零个或多个字符。                  |
| `analyzer`  | 可选      | String   | 用于分析 pattern 文本的解析器。默认为为 <field> 指定的解析器。                                        |
| `use_field` | 可选      | String   | 指定搜索此字段而不是顶层字段。`pattern`使用此字段指定的搜索分词器进行规范化，除非您指定了`analyzer`。 |

> 指定以 \* 或 ? 开头的模式可能会降低搜索性能，因为它增加了匹配词项所需的迭代次数。

### fuzzy 规则

`fuzzy` 规则匹配与提供的词项在指定编辑距离内相似的词项。模糊模式最多可以扩展以匹配 128 个词项。如果模式匹配的词项超过 128 个，则会返回错误。下表列出了 `fuzzy` 规则支持的所有参数。

| 参数             | 必需/可选 | 数据类型 | 描述                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------- | --------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `term`           | 必需      | String   | 要匹配的词项。                                                                                                                                                                                                                                                                                                                                                                                                  |
| `analyzer`       | 可选      | String   | 用于分析 term 文本的解析器。默认为为 <field> 指定的解析器。                                                                                                                                                                                                                                                                                                                                                     |
| `fuzziness`      | 可选      | String   | 在确定词项是否与值匹配时，将一个词转换为另一个词所需的字符编辑次数（插入、删除、替换）。例如， wined 和 wind 之间的距离是 1。有效值是非负整数或 AUTO 。默认值 AUTO 根据每个词的长度选择值，对于大多数用例是一个不错的选择。                                                                                                                                                                                     |
| `transpositions` | 可选      | Boolean  | 将 transpositions 设置为 true （默认值）会将相邻字符的交换添加到 fuzziness 选项的插入、删除和替换操作中。例如，如果 transpositions 为真（交换“n”和“i”），则 wind 和 wnid 之间的距离是 1；如果为假（删除“n”，插入“n”），则距离是 2。如果 transpositions 是 false ，则 rewind 和 wnid 与 wind 的距离相同（2），尽管从更以人为中心的观点来看， wnid 是一个明显的拼写错误。对于大多数用例，默认值是一个不错的选择。 |
| `prefix_length`  | 可选      | Integer  | 模糊匹配时保持不变的开头字符数。默认为 0。                                                                                                                                                                                                                                                                                                                                                                      |
| `use_field`      | 可选      | String   | 指定搜索此字段而不是顶层字段。`term`使用此字段指定的搜索分词器进行规范化，除非您指定了`analyzer`。                                                                                                                                                                                                                                                                                                              |

### regexp 规则

`regexp` 规则使用正则表达式模式匹配词项。正则模式最多可以扩展以匹配 128 个词项。如果模式匹配的词项超过 128 个，则会返回错误。下表列出了 `regexp` 规则支持的所有参数。

| 参数             | 必需/可选 | 数据类型 | 描述                                                                                                                                                       |
| ---------------- | --------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pattern`        | 必需      | String   | 用于匹配词项的正则表达式模式。使用 Lucene 正则表达式语法。                                                                                                 |
| `flags_value`    | 可选      | Integer  | Lucene 正则表达式 flag 的位掩码整数值。可替代字符串形式的 flags。默认启用所有 flag（`ALL`）。                                                               |
| `use_field`      | 可选      | String   | 指定搜索此字段而不是顶层字段。`pattern` 使用此字段指定的搜索分词器进行规范化，除非您指定了其他分词器。                                                      |
| `max_expansions` | 可选      | Integer  | 正则表达式模式可以扩展到的最大词项数。默认为 128。                                                                                                         |

### all_of 规则

`all_of` 规则使用合取（ `AND` ）组合多个规则。下表列出了 `all_of` 规则支持的所有参数。

| 参数        | 必需/可选 | 数据类型           | 描述                                                                                                                                                                                                |
| ----------- | --------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `intervals` | 必需      | 规则对象数组       | 一组用于组合的规则。文档必须匹配所有规则才能在结果中返回。                                                                                                                                          |
| `filter`    | 可选      | 间隔过滤器规则对象 | 用于过滤返回间隔的规则。                                                                                                                                                                            |
| `max_gaps`  | 可选      | Integer            | 匹配词项之间允许的最大位置数。距离超过 max_gaps 的词项不被视为匹配。如果 max_gaps 未指定或设置为 -1 ，无论位置如何，词项都被视为匹配。如果 max_gaps 设置为 0 ，匹配的词项必须相邻出现。默认为 -1 。 |
| `ordered`   | 可选      | Boolean            | 如果 true ，则根据规则生成的间隔应按指定顺序出现。默认为 false 。                                                                                                                                   |

### any_of 规则

`any_of` 规则使用析取（ `OR` ）组合多个规则。文档只需匹配任意一条规则即可返回。下表列出了 `any_of` 规则支持的所有参数。

| 参数        | 必需/可选 | 数据类型           | 描述                                                       |
| ----------- | --------- | ------------------ | ---------------------------------------------------------- |
| `intervals` | 必需      | 规则对象数组       | 一组用于组合的规则。文档只需匹配**任意一条**规则即可返回。 |
| `filter`    | 可选      | 间隔过滤器规则对象 | 用于过滤返回间隔的规则。                                   |

### filter 规则

`filter` 规则用于限制结果。下表列出了 `filter` 规则支持的所有参数。

| 参数               | 必需/可选 | 数据类型 | 描述                                                     |
| ------------------ | --------- | -------- | -------------------------------------------------------- |
| `after`            | 可选      | 查询对象 | 返回出现在指定间隔之后的间隔。             |
| `before`           | 可选      | 查询对象 | 返回出现在指定间隔之前的间隔。   |
| `contained_by`     | 可选      | 查询对象 | 返回被指定间隔所包含的间隔。 |
| `containing`       | 可选      | 查询对象 | 返回包含指定间隔的间隔。       |
| `not_contained_by` | 可选      | 查询对象 | 返回不被指定间隔所包含的间隔。     |
| `not_containing`   | 可选      | 查询对象 | 返回不包含指定间隔的间隔。       |
| `not_overlapping`  | 可选      | 查询对象 | 返回与指定间隔不重叠的间隔。       |
| `overlapping`      | 可选      | 查询对象 | 返回与指定间隔重叠的间隔。       |
| `script`           | 可选      | 查询对象 | 用于匹配文档的脚本。此脚本必须返回 true 或 false 。      |

#### 示例：过滤器

以下查询搜索包含 `pairs` 和 `hash` 的文档，这两个词彼此相距不超过五个位置，并且它们之间不包含 `efficiently` 这个词：

```
POST /testindex/_search
{
  "query": {
    "intervals" : {
      "title" : {
        "match" : {
          "query" : "pairs hash",
          "max_gaps" : 5,
          "filter" : {
            "not_containing" : {
              "match" : {
                "query" : "efficiently"
              }
            }
          }
        }
      }
    }
  }
}
```

返回内容中只包含文档 2：

```
{
  "took": 2,
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
    "max_score": 0.25,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.25,
        "_source": {
          "title": "store key-value pairs in a hash map"
        }
      }
    ]
  }
}
```

#### 示例：脚本过滤器

或者，您可以使用以下变量编写自己的脚本过滤器，并将其与 `intervals` 查询一起使用：

- `interval.start` : 间隔开始的位置（词编号）。
- `interval.end` : 间隔结束的位置（词编号）。
- `interval.gap` : 两个词之间的词数。

例如，以下查询在指定间隔内搜索相邻的 `map` 和 `hash` 这两个词。词的编号从 0 开始，所以在文本 `store key-value pairs in a hash map` 中， `store` 位于位置 0， `key` 位于位置 1 ，以此类推。指定的间隔应从 `a` 之后开始，并在字符串末尾之前结束：

```
POST /testindex/_search
{
  "query": {
    "intervals" : {
      "title" : {
        "match" : {
          "query" : "map hash",
          "filter" : {
            "script" : {
              "source" : "interval.start > 5 && interval.end < 8 && interval.gaps == 0"
            }
          }
        }
      }
    }
  }
}
```

返回内容包含文档 2：

```
{
  "took": 1,
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
    "max_score": 0.5,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.5,
        "_source": {
          "title": "store key-value pairs in a hash map"
        }
      }
    ]
  }
}
```

## 间隔最小化

为确保查询在线性时间内运行， `intervals` 查询会最小化间隔。例如，考虑一个包含文本`a b c d c`的文档。您可以使用以下查询来搜索被 `a` 和 `c` 包含的 `d` ：

```
POST /testindex/_search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "d",
          "filter" : {
            "contained_by" : {
              "match" : {
                "query" : "a c"
              }
            }
          }
        }
      }
    }
  }
}
```

该查询没有返回结果，因为它匹配了前两个词 `a c` ，并且在这两个词之间没有找到 `d` 。


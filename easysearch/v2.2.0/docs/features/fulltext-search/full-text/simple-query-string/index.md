---
title: "Simple Query String 查询"
date: 0001-01-01
summary: "Simple Query String 查询 #  使用 simple_query_string 查询在查询字符串中直接指定由正则表达式分隔的多个参数。简单查询字符串的语法比 query_string 查询宽松，因为它会丢弃字符串中的任何无效部分，并且不会因无效语法而返回错误。
此查询使用简单语法根据特殊运算符解析查询字符串，并将字符串拆分为词项。解析后，查询会独立分析每个词项，然后返回匹配的文档。
相关指南（先读这些） #    全文搜索  Query DSL 基础  以下查询对 title 字段执行模糊搜索：
GET _search { &#34;query&#34;: { &#34;simple_query_string&#34;: { &#34;query&#34;: &#34;\&#34;rises wind the\&#34;~4 | *ising~2&#34;, &#34;fields&#34;: [&#34;title&#34;] } } } 简单字符串语法 #  查询字符串由词项和运算符组成。词项是一个单词（例如，在查询 wind rises 中，词项是 wind 和 rises ）。如果多个词项被引号包围，它们被视为一个短语，其中单词按出现的顺序匹配（例如， &ldquo;wind rises&rdquo; ）。 + 、 | 和 - 等运算符指定用于解释查询字符串中文本的布尔逻辑。
操作符 #  简单查询字符串语法支持以下运算符。
   操作符 描述     + 作为 AND 操作符。   \| 作为 OR 操作符。   * 在词尾使用时，表示前缀查询。   &quot; 将多个词括起来组成短语（例如，&quot;wind rises&quot;）。   (, ) 为优先级包装子句（例如，wind + (rises \| rising)）。   ~n 在词后面使用时（例如，wnid~3），设置 fuzziness。在短语后面使用时，设置 slop。   - 否定该词。    所有前面的操作符都是保留字符。要将其作为原始字符而不是操作符引用，用反斜杠转义它们中的任何一个。在发送 JSON 请求时，使用 \\ 转义保留字符（因为反斜杠字符本身也是保留的，你必须用另一个反斜杠转义反斜杠）。"
---


# Simple Query String 查询

使用 `simple_query_string` 查询在查询字符串中直接指定由正则表达式分隔的多个参数。简单查询字符串的语法比 `query_string` 查询宽松，因为它会丢弃字符串中的任何无效部分，并且不会因无效语法而返回错误。

此查询使用简单语法根据特殊运算符解析查询字符串，并将字符串拆分为词项。解析后，查询会独立分析每个词项，然后返回匹配的文档。

## 相关指南（先读这些）

- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

以下查询对 `title` 字段执行模糊搜索：

```
GET _search
{
  "query": {
    "simple_query_string": {
      "query": "\"rises wind the\"~4 | *ising~2",
      "fields": ["title"]
    }
  }
}
```

## 简单字符串语法

查询字符串由词项和运算符组成。词项是一个单词（例如，在查询 wind rises 中，词项是 wind 和 rises ）。如果多个词项被引号包围，它们被视为一个短语，其中单词按出现的顺序匹配（例如， "wind rises" ）。 + 、 | 和 - 等运算符指定用于解释查询字符串中文本的布尔逻辑。

## 操作符

简单查询字符串语法支持以下运算符。

| 操作符   | 描述                                                                                   |
| -------- | -------------------------------------------------------------------------------------- |
| `+`      | 作为 `AND` 操作符。                                                                    |
| `\|`    | 作为 `OR` 操作符。                                                                     |
| `*`      | 在词尾使用时，表示前缀查询。                                                           |
| `"`      | 将多个词括起来组成短语（例如，`"wind rises"`）。                                     |
| `(`, `)` | 为优先级包装子句（例如，`wind + (rises \| rising)`）。      |
| `~n`     | 在词后面使用时（例如，`wnid~3`），设置 `fuzziness`。在短语后面使用时，设置 `slop`。 |
| `-`      | 否定该词。                                                                             |

所有前面的操作符都是保留字符。要将其作为原始字符而不是操作符引用，用反斜杠转义它们中的任何一个。在发送 JSON 请求时，使用 `\\` 转义保留字符（因为反斜杠字符本身也是保留的，你必须用另一个反斜杠转义反斜杠）。

## 默认操作符

默认操作符是 `OR` （除非你将 `default_operator` 设置为 `AND` ）。默认操作符决定了整体查询行为。例如，考虑一个包含以下文档的索引：

```
PUT /customers/_doc/1
{
  "first_name":"Amber",
  "last_name":"Duke",
  "address":"880 Holmes Lane"
}

PUT /customers/_doc/2
{
  "first_name":"Hattie",
  "last_name":"Bond",
  "address":"671 Bristol Street"
}

PUT /customers/_doc/3
{
  "first_name":"Nanette",
  "last_name":"Bates",
  "address":"789 Madison St"
}


PUT /customers/_doc/4
{
  "first_name":"Dale",
  "last_name":"Amber",
  "address":"467 Hutchinson Court"
}
```

以下查询试图找到地址中包含 `street` 或 `st` 且不包含 `madison` 的文档：

```
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "street st -madison"
    }
  }
}
```

然而，结果不仅包括预期的文档，还包括所有四个文档：

```
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 2.2039728,
    "hits": [
      {
        "_index": "customers",
        "_id": "2",
        "_score": 2.2039728,
        "_source": {
          "first_name": "Hattie",
          "last_name": "Bond",
          "address": "671 Bristol Street"
        }
      },
      {
        "_index": "customers",
        "_id": "3",
        "_score": 1.2039728,
        "_source": {
          "first_name": "Nanette",
          "last_name": "Bates",
          "address": "789 Madison St"
        }
      },
      {
        "_index": "customers",
        "_id": "1",
        "_score": 1,
        "_source": {
          "first_name": "Amber",
          "last_name": "Duke",
          "address": "880 Holmes Lane"
        }
      },
      {
        "_index": "customers",
        "_id": "4",
        "_score": 1,
        "_source": {
          "first_name": "Dale",
          "last_name": "Amber",
          "address": "467 Hutchinson Court"
        }
      }
    ]
  }
}
```

因为默认操作符是 `OR` ，这个查询包括包含 `street` 或 `st` 的文档（文档 2 和 3）以及不包含 `madison` 的文档（文档 1 和 4）。

要正确表达查询意图，请在 `-madison` 前面加上 `+` ：

```
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "street st +-madison"
    }
  }
}
```

或者，将 `AND` 指定为默认操作符，并使用 `street` 和 `st` 的析取操作：

```
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "st|street -madison",
      "default_operator": "AND"
    }
  }
}
```

前面的查询返回文档 2：

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
    "max_score": 2.2039728,
    "hits": [
      {
        "_index": "customers",
        "_id": "2",
        "_score": 2.2039728,
        "_source": {
          "first_name": "Hattie",
          "last_name": "Bond",
          "address": "671 Bristol Street"
        }
      }
    ]
  }
}
```

## 限制操作符

要限制简单查询字符串解析器支持的操作符，请在 `flags` 参数中包含您想要支持的操作符，操作符之间用 `|` 分隔。例如，以下查询仅启用 `OR` 、 `AND` 和 `FUZZY` 操作符：

```
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "bristol | madison +stre~2",
      "flags": "OR|AND|FUZZY"
    }
  }
}
```

下列表格列出了所有可用的操作符标志。

| 标志         | 描述                                                                                |
| ------------ | ----------------------------------------------------------------------------------- |
| `AND` (默认) | 启用 `+` ( `AND` ) 操作符。                                                         |
| `ESCAPE`     | 启用 `\` 作为转义字符。                                                             |
| `FUZZY`      | 启用单词后的 `~n` 操作符，其中 `n` 表示匹配允许的编辑距离。                         |
| `NEAR`       | 启用短语后的 `~n` 操作符，其中 `n` 是允许匹配标记之间的最大位置数。与 `SLOP` 相同。 |
| `NONE`       | 禁用所有操作符。                                                                    |
| `NOT`        | 启用 `-` ( `NOT` ) 操作符。                                                         |
| `OR`         | 启用 `\|` ( `OR` ) 操作符。                                                         |
| `PHRASE`     | 启用 `"` (引号) 用于短语搜索。                                                      |
| `PRECEDENCE` | 启用 `(` 和 `)` (括号) 用于操作符优先级。                                           |
| `PREFIX`     | 启用 `*` (前缀) 操作符。                                                            |
| `SLOP`       | 启用短语后的 `~n` 操作符，其中 `n` 是允许匹配标记之间的最大位置数。与 `NEAR` 相同。 |
| `WHITESPACE` | 启用空格字符作为文本分割的字符。                                                    |

## 通配符表达式

您可以使用 `*` 特殊字符指定通配符表达式，该字符替换零个或多个字符。例如，以下查询搜索所有以 `name` 结尾的字段：

```
GET /customers/_search
{
  "query": {
    "simple_query_string" : {
      "query":    "Amber Bond",
      "fields": [ "*name" ]
    }
  }
}
```

## 加权

使用尖号（`^`）提升运算符来提升字段的相关性得分。[0, 1)范围内的值会降低相关性，而大于 1 的值会增加相关性。默认值为 1。

例如，以下查询搜索 `first_name` 和 `last_name` 字段，并将 `first_name` 字段的匹配项提升 2 倍：

```
GET /customers/_search
{
  "query": {
    "simple_query_string" : {
      "query":    "Amber",
      "fields": [ "first_name^2", "last_name" ]
    }
  }
}
```

## 多位置词元

对于多位置词元，简单查询字符串会创建一个匹配短语查询。因此，如果你指定 `ml, machine learning` 作为同义词并搜索 `ml` ，Easysearch 会搜索 `ml OR "machine learning" `。

或者，您可以使用连词来匹配多位置词元。如果您将 `auto_generate_synonyms_phrase_query` 设置为 `false` ，Easysearch 将搜索 `ml OR (machine AND learning)` 。

例如，以下查询搜索文本 `ml models` 并指定不针对每个同义词自动生成匹配短语查询：

```
GET /testindex/_search
{
  "query": {
    "simple_query_string": {
      "fields": ["title"],
      "query": "ml models",
      "auto_generate_synonyms_phrase_query": false
    }
  }
}
```

对于此查询，Easysearch 创建以下布尔查询： `(ml OR (machine AND learning)) models` 。

## 参数说明

以下表格列出了 `simple_query_string` 查询支持的一级参数。除了 `query` 之外的所有参数都是可选的。

| 参数                                  | 数据类型                                 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`                               | String                                   | 用于搜索的查询字符串语法中可能包含的表达式。必填。                                                                                                                                                                                                                                                                                                                                                                     |
| `analyze_wildcard`                    | Boolean                                  | 指定 Easysearch 是否应尝试分析通配符词项。默认值为 false 。                                                                                                                                                                                                                                                                                                                                                            |
| `analyzer`                            | String                                   | 用于对查询字符串文本进行分词的分词器。默认值为为 default_field 指定的索引时分词器。如果未为 default_field 指定分词器，则 analyzer 为索引的默认分词器。有关 index.query.default_field 的更多信息，请参阅动态索引级索引设置。                                                                                                                                                                                            |
| `auto_generate_synonyms_phrase_query` | Boolean                                  | 指定是否应自动为多词同义词创建 match_phrase 查询。默认值为 true 。                                                                                                                                                                                                                                                                                                                                                     |
| `default_operator`                    | String                                   | 如果查询字符串包含多个搜索词，文档是否需要所有词都匹配（ AND ）或只需一个词匹配（ OR ）才被视为匹配。有效值为：- OR : 字符串 to be 被解释为 to OR be;- AND : 字符串 to be 被解释为 to AND be;默认为 OR 。                                                                                                                                                                                                              |
| `fields`                              | 字符串数组                               | 要搜索的字段列表（例如， "fields": ["title^4", "description"] ）。支持通配符。如果未指定，则默认为 index.query.default_field 设置，该设置默认为 ["*"] 。同时可以搜索的最大字段数由 indices.query.bool.max_clause_count 定义，默认为 1,024。                                                                                                                                                                            |
| `flags`                               | String                                   | 一个以 `\|` 分隔的启用标志字符串（例如，`AND\|OR\|NOT`）。默认为 `ALL`，表示启用所有操作符。设置为 `NONE` 可禁用所有操作符。详情参见上方「限制操作符」章节。                                                                                                                                                                                                                                          |
| `fuzzy_max_expansions`                | 正整数                                   | 查询可以扩展到的最大词数。模糊查询会扩展到与指定距离（ fuzziness ）内的匹配词。然后 Easysearch 尝试匹配这些词。默认值为 50 。                                                                                                                                                                                                                                                                                          |
| `fuzzy_transpositions`                | Boolean                                  | 将 fuzzy_transpositions 设置为 true （默认）会在 fuzziness 选项的插入、删除和替换操作中添加相邻字符的交换。例如，如果 fuzzy_transpositions 为真（交换“n”和“i”），则 wind 和 wnid 之间的距离为 1；如果为假（删除“n”，插入“n”），则距离为 2。如果 fuzzy_transpositions 为假， rewind 和 wnid 与 wind 的距离相同（2），尽管从更以人为中心的观点来看， wnid 是一个明显的拼写错误。对于大多数用例，默认值是一个不错的选择。 |
| `fuzzy_prefix_length`                 | Integer                                  | 模糊匹配时保持不变的开头字符数。默认为 0。                                                                                                                                                                                                                                                                                                                                                                             |
| `lenient`                             | Boolean                                  | 将 lenient 设置为 true 会忽略查询与文档字段之间的数据类型不匹配。例如， "8.2" 的查询字符串可以匹配类型为 float 的字段。默认值为 false 。                                                                                                                                                                                                                                                                               |
| `minimum_should_match`                | 正整数或负整数、正百分比或负百分比、组合 | 如果查询字符串包含多个搜索词并且你使用 or 运算符，文档被考虑为匹配所需的匹配词数。例如，如果 minimum_should_match 为 2， wind often rising 不匹配 The Wind Rises. 如果 minimum_should_match 为 1 ，则匹配。详情请参阅 Minimum should match。                                                                                                                                                                           |
| `quote_field_suffix`                  | String                                   | 此选项支持使用与非精确匹配不同的分析方法来搜索精确匹配（用引号括起来）。例如，如果 `quote_field_suffix` 是 `.exact`，而你搜索 `"lightly"` 在 `title` 字段中，Easysearch 会在 `title.exact` 字段中搜索。这个第二个字段可能使用不同的类型（例如，`keyword` 而不是 `text`）或不同的分词器。                                                                                                                           |
| `boost`                               | Float                                    | 用于调整该查询的相关性得分权重。默认值为 `1.0`。                                                                                                                                                                                                                                                                                                                                                                       |

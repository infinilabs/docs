---
title: "Query String 查询"
date: 0001-01-01
summary: "Query String 查询 #  query_string 查询根据查询字符串语法解析查询字符串。它提供了创建强大而简洁的查询的功能，这些查询可以包含通配符并搜索多个字段。
相关指南（先读这些） #    全文搜索  Query DSL 基础  查询字符串语法 — 通配符、模糊、范围、布尔等完整语法参考   使用注意：使用 query_string 查询的搜索不会返回嵌套文档。要搜索嵌套字段，请使用 nested 查询。
  语法严格性：查询字符串查询具有严格的语法，在语法无效时会返回错误。因此，它不适合搜索框应用程序。对于不太严格的替代方案，可以考虑使用 simple_query_string 查询。如果你不需要查询语法支持，使用 match 查询。
 参考样例 #  运行以下搜索时， query_string 查询会将 (new york city) OR (big apple) 拆分为两部分： new york city 和 big apple 。 content 字段的分词器随后会分别将每个部分转换为标记，然后返回匹配的文档。由于查询语法不使用空格作为运算符，因此 new york city 会按原样传递给分词器。
GET /_search { &#34;query&#34;: { &#34;query_string&#34;: { &#34;query&#34;: &#34;(new york city) OR (big apple)&#34;, &#34;default_field&#34;: &#34;content&#34; } } } 参数说明 #  下表列出了 query_string 查询支持的参数。除 query 外，所有参数都是可选的。"
---


# Query String 查询

`query_string` 查询根据查询字符串语法解析查询字符串。它提供了创建强大而简洁的查询的功能，这些查询可以包含通配符并搜索多个字段。

## 相关指南（先读这些）

- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [查询字符串语法]({{< relref "/docs/features/query-dsl/query-parser-syntax.md" >}}) — 通配符、模糊、范围、布尔等完整语法参考

> **使用注意**：使用 query_string 查询的搜索不会返回嵌套文档。要搜索嵌套字段，请使用 nested 查询。

> **语法严格性**：查询字符串查询具有严格的语法，在语法无效时会返回错误。因此，它不适合搜索框应用程序。对于不太严格的替代方案，可以考虑使用 simple_query_string 查询。如果你不需要查询语法支持，使用 match 查询。

## 参考样例
运行以下搜索时， `query_string` 查询会将 `(new york city) OR (big apple)` 拆分为两部分： `new york city` 和 `big apple` 。 `content` 字段的分词器随后会分别将每个部分转换为标记，然后返回匹配的文档。由于查询语法不使用空格作为运算符，因此 `new york city` 会按原样传递给分词器。

```
GET /_search
{
  "query": {
    "query_string": {
      "query": "(new york city) OR (big apple)",
      "default_field": "content"
    }
  }
}
```


## 参数说明

下表列出了 `query_string` 查询支持的参数。除 `query` 外，所有参数都是可选的。

| 参数                                  | 数据类型                                 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------- | ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`                               | String                                   | 用于搜索的查询字符串语法中可能包含的表达式。必填。                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `allow_leading_wildcard`              | Boolean                                  | 指定是否允许 \* 和 ? 作为搜索词的首字符。默认为 true 。                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `analyze_wildcard`                    | Boolean                                  | 指定 Easysearch 是否应尝试分析通配符词项。默认值为 false 。                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `analyzer`                            | String                                   | 用于对查询字符串文本进行分词的分词器。默认值为为 default_field 指定的索引时分词器。如果未为 default_field 指定分词器，则 analyzer 为索引的默认分词器。有关 index.query.default_field 的更多信息，请参阅动态索引级索引设置。                                                                                                                                                                                                                                                                                    |
| `auto_generate_synonyms_phrase_query` | Boolean                                  | 指定是否自动为多词同义词创建匹配短语查询。例如，如果你将 ba, batting average 指定为同义词并搜索 ba ，Easysearch 会搜索 ba OR "batting average" （如果此选项为 true ）或 ba OR (batting AND average) （如果此选项为 false ）。默认值为 true 。                                                                                                                                                                                                                                                                  |
| `boost`                               | float                                    | 通过给定的乘数提升子句的权重。适用于在复合查询中对子句进行加权。[0, 1) 范围内的值会降低相关性，而大于 1 的值会增加相关性。默认值为 1 。                                                                                                                                                                                                                                                                                                                                                                        |
| `default_field`                       | String                                   | 如果查询字符串中未指定字段，则在此字段中搜索。支持通配符。默认值为 `index.query.default_field` 索引设置中指定的值。默认情况下，`index.query.default_field` 是 `*`，这意味着提取所有适用于 term 查询的字段并过滤元数据字段。如果未指定 prefix，则提取的字段将组合成一个查询。符合条件的字段不包括嵌套文档。搜索所有符合条件的字段可能是一个资源密集型操作。`indices.query.bool.max_clause_count` 搜索设置定义了字段数量和一次可以查询的词数乘积的最大值。`indices.query.bool.max_clause_count` 的默认值为 1,024。 |
| `default_operator`                    | String                                   | 如果查询字符串包含多个搜索词，文档是否需要所有词都匹配（ AND ）或只需一个词匹配（ OR ）才被视为匹配。有效值为：- OR : 字符串 to be 被解释为 to OR be;- AND : 字符串 to be 被解释为 to AND be;默认为 OR 。                                                                                                                                                                                                                                                                                                      |
| `enable_position_increments`          | Boolean                                  | 当 true 时，生成的查询会考虑位置增量。当停用词被移除时，留下不想要的“间隙”时，此设置很有用。默认值为 true 。                                                                                                                                                                                                                                                                                                                                                                                                   |
| `fields`                              | 字符串数组                               | 要搜索的字段列表（例如，`"fields": ["title^4", "description"]`）。支持通配符。如果未指定，则默认为 `index.query.default_field` 设置，该设置默认为 `["*"]`。                                                                                                                                                                                                                                                                                                                                                      |
| `fuzziness`                           | String                                   | 在确定词项是否与值匹配时，将一个词转换为另一个词所需的字符编辑次数（插入、删除、替换）。例如， wined 和 wind 之间的距离是 1。有效值是非负整数或 AUTO 。默认值 AUTO 根据每个词的长度选择值，对于大多数用例是一个不错的选择。                                                                                                                                                                                                                                                                                    |
| `fuzzy_max_expansions`                | 正整数                                   | 查询可以扩展到的最大词数。模糊查询会扩展到与指定距离（ fuzziness ）内的匹配词。然后 Easysearch 尝试匹配这些词。默认值为 50 。                                                                                                                                                                                                                                                                                                                                                                                  |
| `fuzzy_transpositions`                | Boolean                                  | 将 fuzzy_transpositions 设置为 true （默认）会在 fuzziness 选项的插入、删除和替换操作中添加相邻字符的交换。例如，如果 fuzzy_transpositions 为真（交换“n”和“i”），则 wind 和 wnid 之间的距离为 1；如果为假（删除“n”，插入“n”），则距离为 2。如果 fuzzy_transpositions 为假， rewind 和 wnid 与 wind 的距离相同（2），尽管从更以人为中心的观点来看， wnid 是一个明显的拼写错误。对于大多数用例，默认值是一个不错的选择。                                                                                         |
| `lenient`                             | Boolean                                  | 将 lenient 设置为 true 会忽略查询与文档字段之间的数据类型不匹配。例如， "8.2" 的查询字符串可以匹配类型为 float 的字段。默认值为 false 。                                                                                                                                                                                                                                                                                                                                                                       |
| `max_determinized_states`             | 正整数                                   | Lucene 可以为包含正则表达式（例如 `"query": "/wind.+?/"` ）的查询字符串创建的最大“状态”数量（复杂性的度量），数值越大允许使用更多内存的查询。默认值为 10,000。                                                                                                                                                                                                                                                                                                                                                 |
| `minimum_should_match`                | 正整数或负整数、正百分比或负百分比、组合 | 如果查询字符串包含多个搜索词并且你使用 or 运算符，文档被考虑为匹配所需的匹配词数。例如，如果 minimum_should_match 为 2， wind often rising 不匹配 The Wind Rises. 如果 minimum_should_match 为 1 ，则匹配。详情请参阅 Minimum should match。                                                                                                                                                                                                                                                                   |
| `phrase_slop`                         | Integer                                  | 允许在匹配的词之间出现的最大词数。如果 phrase_slop 为 2，则短语中匹配的词之间最多允许两个词。调换顺序的词有 2 的容差。默认值为 0 （精确短语匹配，其中匹配的词必须相邻）。                                                                                                                                                                                                                                                                                                                                      |
| `quote_analyzer`                      | String                                   | 用于在查询字符串中分词引号内文本的分词器。覆盖引号文本的 analyzer 参数。默认值为 default_field 指定的 search_quote_analyzer 。                                                                                                                                                                                                                                                                                                                                                                                 |
| `quote_field_suffix`                  | String                                   | 此选项支持使用与非精确匹配不同的分析方法来搜索精确匹配（用引号括起来）。例如，如果 quote_field_suffix 是 .exact ，而你搜索 \"lightly\" 在 title 字段中，Easysearch 会在 lightly 字段中搜索单词 title.exact 。这个第二个字段可能使用不同的类型（例如， keyword 而不是 text ）或不同的分词器。                                                                                                                                                                                                                   |
| `rewrite`                             | String                                   | 确定 Easysearch 如何重写和评分多词查询。有效值为 constant_score 、 scoring_boolean 、 constant_score_boolean 、 top_terms_N 、 top_terms_boost_N 和 top_terms_blended_freqs_N 。默认值为 constant_score 。                                                                                                                                                                                                                                                                                                     |
| `tie_breaker`                         | Float（0.0~1.0）                         | 当查询使用多个字段时，`tie_breaker` 控制非最佳匹配字段对评分的贡献。`0.0` 只取最佳匹配字段得分，`1.0` 将所有匹配字段得分相加。默认值为 `0.0`。                                                                                                                                                                                                                                                                                                                                                                  |
| `time_zone`                           | String                                   | 指定从 UTC 偏移所需时区的小时数。如果查询字符串包含日期范围，则需要指定时区偏移数字。例如，对于包含日期范围 `"query": "wind rises release_date[2012-01-01 TO 2014-01-01]"` 的查询，请设置 `time_zone: "-08:00"`。默认时区格式为 UTC。                                                                                                                                                                                                                                                                      |

## query string 字符串查询语法

字符串查询 (query string) 语法基于 Apache Lucene 查询语法。下面会介绍相关的语法，详细的使用可以[参考这里](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)

在以下情况下，您可以使用查询字符串语法：

在一个 query_string 查询中，例如：

```
GET _search
 {
   "query": {
     "query_string": {
       "query": "the wind AND (rises OR rising)"
     }
   }
 }
```

如果你使用 HTTP 请求查询参数进行搜索，例如：

```
 GET _search?q=wind

```

字符串查询由词项和运算符组成。词项是一个单词（例如，在查询 `wind rises` 中，词项是 `wind` 和 `rises` ）。如果多个词项被引号包围，它们被视为一个短语，其中单词按出现的顺序匹配（例如， "wind rises" ）。运算符（如 `OR` 、 `AND` 和 `NOT` ）指定用于解释查询字符串中文本的布尔逻辑。

本节中的示例使用一个包含以下映射和文档的索引：

```
PUT /testindex
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}

PUT /testindex/_doc/1
{
  "title": "The wind rises"
}

PUT /testindex/_doc/2
{
  "title": "Gone with the wind",
  "description": "A 1939 American epic historical film"
}

PUT /testindex/_doc/3
{
  "title": "Windy city"
}

PUT /testindex/_doc/4
{
  "article title": "Wind turbines"
}
```

### 保留字符

以下是查询字符串查询的保留字符列表：

```
+, -, =, &&, ||, >, <, !, (, ),{, }, [, ], ^, ", ~, *, ?, :, \, /

```

> 使用反斜杠（ \ ）转义保留字符。在发送 JSON 请求时，使用双反斜杠（ \\ ）转义保留字符（因为反斜杠字符本身也是保留字符，你必须用另一个反斜杠来转义反斜杠）。

例如，要搜索表达式 `2*3` ，请指定查询字符串： `2\\*3` 。

```
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: 2\\*3"
    }
  }
}
```

> `>` 和 `<` 符号不能转义。它们被视为范围查询。

### 空白字符和空查询

空白字符不被视为操作符。如果查询字符串为空或只包含空白字符，查询不会返回结果。

### 字段名

在冒号前指定字段名。下表包含带有字段名的示例查询。

| query_string 查询中的查询 | 文档匹配的标准                                                                                  | 从 testindex 索引中匹配文档 |
| ------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------- |
| `title: wind`             | title 字段包含单词 wind 。                                                                      | 1、2                        |
| `title: (wind OR windy)`  | title 字段包含单词 wind 或单词 windy 。                                                         | 1、2、3                     |
| `title: \"wind rises\"`    | title 字段包含短语 wind rises 。使用反斜杠转义引号。                                            | 1                           |
| `article\\ title: wind`   | article title 字段包含单词 wind 。用反斜杠转义空格字符。                                        | 4                           |
| `title.\\*: rise`         | 以 title. 开头的每个字段（在这个例子中是 title.english ）都包含单词 rise 。用反斜杠转义通配符。 | 1                           |
| `_exists_: description`   | The field description exists. 字段 description 存在。                                           | 2                           |

### 通配符表达式

您可以使用特殊字符指定通配符表达式： `?` 替换单个字符， `*` 替换零个或多个字符。

#### 参考样例

以下查询搜索标题包含 `gone` 的词，并且描述包含以 `hist` 开头的词：

```
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: gone AND description: hist*"
    }
  }
}
```

> 通配符查询可能会使用大量内存，这可能会降低性能。词首的通配符（例如 \*cal ）是最昂贵的，因为匹配这些通配符的文档需要检查索引中的所有词。要禁用首部通配符，请将 allow_leading_wildcard 设置为 false 。

为了效率，纯通配符如 `*` 会被重写为 `exists` 查询。因此， `description: * `通配符会匹配 `description` 字段包含空值的文档，但不会匹配 `description` 字段缺失或具有 `null` 值的文档。

如果你将 `analyze_wildcard` 设置为 `true` ，Easysearch 将分析以 `*` 结尾的查询（例如 `hist*` ）。因此，Easysearch 将通过在第一个 n-1 个词上精确匹配，并在最后一个词上前缀匹配，来构建一个包含结果词的布尔查询。

### 正则表达式

要在查询字符串中指定正则表达式模式，用正斜杠 ( `/` ) 将其包围，例如 `title: /w[a-z]nd/` 。

> allow_leading_wildcard 参数不适用于正则表达式。例如，查询字符串如 /.\*d/ 将检查索引中的所有词项。

### 模糊性

你可以使用 `~` 操作符运行模糊查询，例如 `title: rise~` 。

该查询会搜索包含与搜索词相似（在最大允许编辑距离内）的词项的文档。编辑距离定义为 Damerau-Levenshtein 距离，它测量将一个词项转换为另一个词项所需的单字符更改次数（插入、删除、替换或转换）。

默认的编辑距离为 2，应该能捕获 80%的拼写错误。要更改默认编辑距离，请在 `~` 操作符后指定新的编辑距离。例如，要将编辑距离设置为 1 ，请使用查询 `title: rise~1` 。

> 不要混合使用模糊和通配符操作符。如果你同时指定了模糊和通配符操作符，其中一个操作符将不会被应用。例如，如果你可以搜索 wnid*~1 ，通配符操作符 * 将会被应用，但模糊操作符 ~1 将不会被应用。

### 邻近查询

邻近查询不需要搜索短语中的词语按指定顺序排列。它允许短语中的词语以不同的顺序出现或被其他词语分隔。邻近查询指定短语中词语的最大编辑距离。例如，以下查询在匹配指定短语中的词语时允许编辑距离为 4：

```
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: \"wind gone\"~4"
    }
  }
}
```

当 Easysearch 匹配文档时，文档中的词语与查询中指定的词语顺序越接近（即编辑距离越小），该文档的相关性得分就越高。

### 范围查询

要为数值、字符串或日期字段指定范围，请使用方括号（ `[min TO max]` ）表示包含范围，使用花括号（ `{min TO max}` ）表示排除范围。您也可以混合使用方括号和花括号来包含或排除下限和上限（例如， `{min TO max]` ）。

日期范围必须使用您在映射包含日期的字段时使用的格式提供。有关支持的日期格式的更多信息，请参阅格式。

下表提供了范围语法示例。

| 数据类型 | 查询内容                                                             | 字符串查询                                                                                        |
| -------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `Numeric`  | 账户号码在 1 到 15（含）范围内的文档。                               | `account_number: [1 TO 15]` 或 `account_number: (>=1 AND <=15)` 或 `account_number: (+>=1 +<=15)` |
|          | 账户号码为 15 及以上的文档。                                         | `account_number: [15 TO *]` 或 `account_number: >=15` （注意 >= 符号后没有空格）                  |
| `String`   | 姓氏从 Bates（包含）到 Duke（不包含）的文档。                        | [`Bates TO Duke}` 或 `lastname: (>=Bates AND <Duke)  `                                            |
|          | 文档中姓氏在字母顺序上位于 Bates 之前。                              | ` lastname: { * TO Bates}` 或 l`astname: <Bates` （注意 < 符号后没有空格）                        |
| `Date`     | 发布日期在 2023 年 3 月 21 日至 2023 年 9 月 25 日（含）之间的文档。 | `release_date: [03/21/2023 TO 09/25/2023] `                                                       |

> 作为在查询字符串中指定范围的一种替代方法，您可以使用范围查询，它提供了更可靠的语法。

### 加权

使用尖号（`^`）提升运算符来通过乘数提升文档的相关性分数。[0, 1)范围内的值会降低相关性，而大于 1 的值会增加相关性。默认值为 1。

下表提供了提升示例。

| 类型     | 描述                                                                          | 字符串查询                  |
| -------- | ----------------------------------------------------------------------------- | --------------------------- |
| 词频提升 | 查找包含词 street 的所有地址，并提升包含词 Madison 的地址的权重。             | `address: Madison^2 street` |
| 短语提升 | 查找标题包含短语 wind rises 的文档，并提升其权重至 2 倍。                     | `title: \"wind rises\"^2`   |
|          | 查找标题包含 wind rises 的文档，并将包含短语 wind rises 的文档提升 2 倍权重。 | `title: (wind rises)^2`     |

### 布尔运算符

当你在查询中提供搜索词时，默认情况下，查询会返回包含至少一个所提供词的文档。你可以使用 `default_operator` 参数为所有词指定运算符。因此，如果你将 `default_operator` 设置为 `AND` ，所有词都将被要求；而如果你将其设置为 `OR` ，所有词都将被视为可选。

#### + 和 - 运算符

如果你需要更精细地控制必需和可选的词项，可以使用 `+` 和 `-` 操作符。 `+` 操作符会使它后面的词项成为必需的，而 `-` 操作符会排除它后面的词项。

例如，在查询字符串中， `title: (gone +wind -turbines)` 指定 `gone` 是可选的， `wind` 必须存在，而 `turbines` 则不能存在于匹配文档的标题中：

```
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: (gone +wind -turbines)"
    }
  }
}
```

查询返回了两个匹配的文档：

```
{
  "_index": "testindex",
  "_id": "2",
  "_score": 1.3159468,
  "_source": {
    "title": "Gone with the wind",
    "description": "A 1939 American epic historical film"
  }
},
{
  "_index": "testindex",
  "_id": "1",
  "_score": 0.3438858,
  "_source": {
    "title": "The wind rises"
  }
}
```

前面的查询等同于以下布尔查询：

```
GET testindex/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "wind"
        }
      },
      "should": {
        "match": {
          "title": "gone"
        }
      },
      "must_not": {
        "match": {
          "title": "turbines"
        }
      }
    }
  }
}
```

#### 传统布尔运算符

或者，您可以使用以下布尔运算符： `AND` 、`&&` 、 `OR` 、 `||` 、 `NOT` 、 `!` 。但是，这些运算符不遵循优先级规则，因此在使用多个布尔运算符时，您必须使用括号来指定优先级。例如，查询字符串 `title: (gone +wind -turbines)` 可以使用布尔运算符重写如下：

运行以下包含重写查询字符串的查询：

```
GET testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: ((gone AND wind) OR wind) AND NOT turbines"
    }
  }
}
```

该查询返回的结果与使用 `+` 和 `-` 运算符的查询相同。但是请注意，匹配文档的相关性分数与之前的结果不同：

```
{
  "_index": "testindex",
  "_id": "2",
  "_score": 1.6166971,
  "_source": {
    "title": "Gone with the wind",
    "description": "A 1939 American epic historical film"
  }
},
{
  "_index": "testindex",
  "_id": "1",
  "_score": 0.3438858,
  "_source": {
    "title": "The wind rises"
  }
}
```

### 分组

使用括号将多个子句或词项组合为子查询。例如，以下查询搜索包含 `gone` 或 `rises` 的文档，这些文档必须在标题中包含 `wind` ：

```
GET testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: (gone OR rises) AND wind"
    }
  }
}
```

结果包含两个匹配的文档：

```
{
  "_index": "testindex",
  "_id": "1",
  "_score": 1.5046883,
  "_source": {
    "title": "The wind rises"
  }
},
{
  "_index": "testindex",
  "_id": "2",
  "_score": 1.3159468,
  "_source": {
    "title": "Gone with the wind",
    "description": "A 1939 American epic historical film"
  }
}

```

您也可以使用分组来提升子查询结果或针对指定字段，例如 `title:(gone AND wind) description:(historical film)^2` 。

### 搜索多个字段

要搜索多个字段，请使用 `fields` 参数。当你提供 `fields` 参数时，查询会被重写为 `field_1: query OR field_2: query ...` 。

例如，以下查询会在 `title` 和 `description` 字段中搜索 `wind` 或 `film` ：

```
GET testindex/_search
{
  "query": {
    "query_string": {
      "fields": [ "title", "description" ],
      "query": "wind AND film"
    }
  }
}
```

前面的查询与不提供 `fields` 参数的以下查询等效：

```
GET testindex/_search
{
  "query": {
    "query_string": {
      "query": "(title:wind OR description:wind) AND (title:film OR description:film)"
    }
  }
}
```

#### 搜索字段中的多个子字段

要搜索某个字段的全部内部字段，可以使用通配符。例如，要搜索 `address` 字段内的所有子字段，可以使用以下查询：

```
GET /testindex/_search
{
  "query": {
    "query_string" : {
      "fields" : ["address.*"],
      "query" : "New AND (York OR Jersey)"
    }
  }
}
```

前面的查询与以下不提供 `fields` 参数的查询等效（注意 `*` 使用 `\\` 转义）：

```
GET /testindex/_search
{
  "query": {
    "query_string" : {
      "query" : "address.\\*: New AND (York OR Jersey)"
    }
  }
}
```

#### 子字段加权

每个搜索词生成的子查询使用 `dis_max` 查询与 `tie_breaker` 结合。要提升单个字段的权重，使用 `^` 操作符。例如，以下查询将 `title` 字段的权重提升 2 倍：

```
GET testindex/_search
{
  "query": {
    "query_string": {
      "fields": [ "title^2", "description" ],
      "query": "wind AND film"
    }
  }
}
```

要提升字段的所有子字段，在通配符后指定提升操作符：

```
GET /testindex/_search
{
  "query": {
    "query_string" : {
      "fields" : ["work_address", "address.*^2"],
      "query" : "New AND (York OR Jersey)"
    }
  }
}
```

#### 多字段搜索参数

在搜索多个字段时，您可以将额外的可选参数 `type` 传递给 `query_string` 查询。

| 参数   | 数据类型 | 描述                                                                                                                                                                                                  |
| ------ | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type` | String   | 确定 Easysearch 如何执行查询和评分结果。有效值为 best_fields 、 bool_prefix 、 most_fields 、 cross_fields 、 phrase 和 phrase_prefix 。默认值为 best_fields 。有关有效值的说明，请参阅多值查询类型。 |

### query_string 查询中的同义词

`query_string` 查询支持使用 `synonym_graph` 令牌过滤器进行多词同义词扩展。如果你使用 `synonym_graph` 令牌过滤器，Easysearch 会为每个同义词创建一个匹配短语查询。

`auto_generate_synonyms_phrase_query` 参数指定是否为多词同义词自动创建匹配短语查询。默认情况下， `auto_generate_synonyms_phrase_query` 为 true ，因此如果你指定 `ml, machine learning` 为同义词并搜索 `ml` ，Easysearch 会搜索 `ml OR "machine learning"` 。

或者，您可以使用连词来匹配多词同义词。如果您将 `auto_generate_synonyms_phrase_query` 设置为 `false` ，Easysearch 将搜索 `ml OR (machine AND learning)` 。

例如，以下查询搜索文本 `ml models` 并指定不针对每个同义词自动生成匹配短语查询：

```
GET /testindex/_search
{
  "query": {
    "query_string": {
      "default_field": "title",
      "query": "ml models",
      "auto_generate_synonyms_phrase_query": false
    }
  }
}
```

对于此查询，Easysearch 创建以下布尔查询： `(ml OR (machine AND learning)) models` 。

### Minimum should match 最小匹配

`query_string` 查询会在每个运算符周围分割查询，并为整个输入创建一个布尔查询。 `minimum_should_match` 参数指定文档必须匹配的最小词数才能在搜索结果中返回。例如，以下查询指定 `description` 字段必须至少匹配两个词才能为每个搜索结果返回：

```
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "description"
      ],
      "query": "historical epic film",
      "minimum_should_match": 2
    }
  }
}
```

对于此查询，Easysearch 创建以下布尔查询： `(description:historical description:epic description:film)~2` 。

#### 最小值应与多个字段匹配

如果你在 `query_string` 查询中指定了多个字段，Easysearch 会为指定的字段创建 `dis_max` 查询。如果你没有显式指定查询项的操作符，整个查询文本被视为一个子句。Easysearch 使用这个单一子句为每个字段构建查询。最终的布尔查询包含一个对应于所有字段 `dis_max` 查询的单一子句，因此 `minimum_should_match` 参数不适用。

例如，在以下查询中， `historical epic heroic` 被视为一个子句：

```
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "description"
      ],
      "query": "historical epic heroic",
      "minimum_should_match": 2
    }
  }
}
```

对于此查询，Easysearch 创建以下布尔查询： `((title:historical title:epic title:heroic) | (description:historical description:epic description:heroic))` 。

如果你在查询词中添加显式操作符（ `AND` 或 `OR` ），每个词被视为一个独立的子句，可以应用 `minimum_should_match` 参数。例如，在以下查询中， `historical` 、 `epic` 和 `heroic` 被视为独立的子句：

```
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "description"
      ],
      "query": "historical OR epic OR heroic",
      "minimum_should_match": 2
    }
  }
}
```

对于此查询，Easysearch 创建以下布尔查询： `((title:historical | description:historical) (description:epic | title:epic) (description:heroic | title:heroic))~2` 。该查询匹配至少三个子句中的两个。每个子句都表示在每个词项的 `title` 和 `description` 字段上的 `dis_max` 查询。

或者，为了确保可以应用 `minimum_should_match` ，你可以将 `type` 参数设置为 `cross_fields` 。这表示在分析输入文本时，具有相同分词器的字段应该被组合在一起：

```
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "description"
      ],
      "query": "historical epic heroic",
      "type": "cross_fields",
      "minimum_should_match": 2
    }
  }
}
```

对于此查询，Easysearch 创建以下布尔查询： `((title:historical | description:historical) (description:epic | title:epic) (description:heroic | title:heroic))~2` 。

然而，如果你使用不同的分词器，必须在查询中使用显式操作符，以确保 `minimum_should_match` 参数应用于每个词。


## 注意
> 查询字符串查询可能会在内部转换为前缀查询。如果 search.allow_expensive_queries 设置为 false ，则不会执行前缀查询。如果 index_prefixes 被启用，则忽略 search.allow_expensive_queries 设置，并构建并执行优化后的查询。


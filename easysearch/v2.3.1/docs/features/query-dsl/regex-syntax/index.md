---
title: "正则表达式语法"
date: 0001-01-01
summary: "正则表达式语法 #  正则表达式（regex）是一种使用特殊符号和运算符定义搜索模式的方法。这些模式允许你在字符串中匹配字符序列。
在 Easysearch 中，你可以在以下查询类型中使用正则表达式：
 regexp query_string   注意：Easysearch 使用 Apache Lucene 正则表达式引擎，该引擎有自己的语法和限制。它不使用 Perl 兼容正则表达式（PCRE），因此某些熟悉的正则表达式功能可能会表现不同或不受支持。
 相关指南（先读这些） #    部分匹配  正则查询  字符串查询  查询字符串语法 — 通配符、模糊、范围、布尔等完整语法参考  在 regexp 和 query_string 查询之间进行选择 #  regexp 和 query_string 查询都支持正则表达式，但它们的行为不同，适用于不同的使用场景。
   特性 regexp 查询 query_string 查询     模式匹配 正则表达式模式必须匹配整个字段值 正则表达式模式可以匹配字段中的任何部分   flags 支持 flags 启用可选正则表达式运算符 flags 不支持   查询类型 词级查询（未评分） 全文检索（评分和解析）   最佳使用场景 对关键字或精确字段进行严格模式匹配 使用支持正则表达式模式的灵活查询字符串在分析字段中进行搜索   复杂查询组合 仅限于正则表达式模式 支持 AND 、 OR 、通配符、字段、提升值以及其他功能。参见查询字符串查询。    保留字符 #  Lucene 的正则表达式引擎支持所有 Unicode 字符。然而，以下字符被视为特殊运算符："
---


# 正则表达式语法

正则表达式（regex）是一种使用特殊符号和运算符定义搜索模式的方法。这些模式允许你在字符串中匹配字符序列。

在 Easysearch 中，你可以在以下查询类型中使用正则表达式：

- `regexp`
- `query_string`

> **注意**：Easysearch 使用 Apache Lucene 正则表达式引擎，该引擎有自己的语法和限制。它不使用 Perl 兼容正则表达式（PCRE），因此某些熟悉的正则表达式功能可能会表现不同或不受支持。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [正则查询]({{< relref "/docs/features/query-dsl/term-based-query/regexp.md" >}})
- [字符串查询]({{< relref "/docs/features/fulltext-search/full-text/query-string.md" >}})
- [查询字符串语法]({{< relref "/docs/features/query-dsl/query-parser-syntax.md" >}}) — 通配符、模糊、范围、布尔等完整语法参考

## 在 regexp 和 query_string 查询之间进行选择

`regexp` 和 `query_string` 查询都支持正则表达式，但它们的行为不同，适用于不同的使用场景。

| 特性         | regexp 查询                        | query_string 查询                                                       |
| ------------ | ---------------------------------- | ----------------------------------------------------------------------- |
| 模式匹配     | 正则表达式模式必须匹配整个字段值   | 正则表达式模式可以匹配字段中的任何部分                                  |
| flags 支持   | flags 启用可选正则表达式运算符     | flags 不支持                                                            |
| 查询类型     | 词级查询（未评分）                 | 全文检索（评分和解析）                                                  |
| 最佳使用场景 | 对关键字或精确字段进行严格模式匹配 | 使用支持正则表达式模式的灵活查询字符串在分析字段中进行搜索              |
| 复杂查询组合 | 仅限于正则表达式模式               | 支持 AND 、 OR 、通配符、字段、提升值以及其他功能。参见查询字符串查询。 |

## 保留字符

Lucene 的正则表达式引擎支持所有 Unicode 字符。然而，以下字符被视为特殊运算符：

```
. ? + * | { } [ ] ( ) " \
```

根据启用的 flags 指定的可选运算符，以下字符也可能被保留：

```
@ & ~ < >
```

要字面匹配这些字符，可以用反斜杠（ `\` ）转义它们，或者将整个字符串用双引号括起来：

- `\&` : 匹配字面量 `&`
- `\\` : 匹配字面量反斜杠 ( `\` )
- `"hello@world"` ：匹配整个字符串 `hello@world`

## 标准正则表达式运算符

Lucene 支持一组核心的正则表达式运算符：

- `.` – 匹配任何单个字符。示例： `f.n` 匹配 `f` 后跟任何字符然后是 `n` （例如 `fan` 或 `fin` ）。
- `?` – 匹配零个或一个前面的字符。示例： `colou?r` 匹配 `color` 和 `colour` 。
- `+` – 匹配一个或多个前面的字符。示例： `go+` 匹配 `g` 后面跟着一个或多个 `o` （ `go` ， `goo` ， `gooo` ，等等）。
- `*` – 匹配零个或多个前面的字符。示例： `lo*se` 匹配 `l` 后面跟着零个或多个 `o` ，然后是 `se` （ `lse` ， `lose` ， `loose` ， `loooose` ，等等）。
- `{min,max}` – 匹配特定的重复次数范围。如果 `max` 被省略，则匹配的字符数量没有上限。示例： `x{3}` 匹配正好 3 个 `x` （ `xxx` ）； `x{2,4}` 匹配 2 到 4 个 `x` （ `xx` ， `xxx` ，或 `xxxx` ）； `x{3,}` 匹配 3 个或更多 `x` （ `xxx` ， `xxxx` ， `xxxxx` ，等等）。
- `|` – 充当逻辑 `OR` 。例如： `apple|orange` 匹配 `apple` 或 `orange` 。
- `( )` – 将字符分组为子模式。例如： `ab(cd)?` 匹配 `ab` 和 `abcd` 。
- `[ ]` – 匹配集合或范围内的一个字符。例如： `[aeiou]` 匹配任何元音。
  - `-` – 当置于括号内时，表示范围，除非被转义或位于括号内的第一个字符。例如： `[a-z]` 匹配任何小写字母； `[-az]` 匹配 `-` 、 `a` 或 `z` ； `[a\\-z]` 匹配 `a` 、 `-` 或 `z` 。
  - `^` – 当置于方括号内时，充当逻辑 `NOT` ，否定一个字符范围或集合中的任意字符。示例： `[^az]` 匹配除 `a` 或 `z` 之外的任意字符； `[^a-z]` 匹配除小写字母之外的任意字符； `[^-az]` 匹配除 `-` 、 `a` 和 `z` 之外的任意字符； `[^a\\-z]` 匹配除 `a` 、 `-` 和 `z` 之外的任意字符。

## 可选运算符

你可以使用 `flags` 参数启用额外的正则表达式运算符。多个标志之间用 `|` 分隔。

以下是可以使用的标志：

- `ALL` (默认) – 启用所有可选运算符。
- `COMPLEMENT` – 启用 `~` ，该标志会否定最短匹配的表达式。示例： `d~ef` 匹配 `dgf` 、 `dxf` ，但不匹配 `def` 。
- `INTERSECTION` – 启用 `&` 作为 `AND` 逻辑运算符。示例： `ab.+&.+cd` 匹配包含 `ab` 开头和 `cd` 结尾的字符串。
- `INTERVAL` – 启用 `<min-max>` 语法来匹配数字范围。示例： `id<10-12>` 匹配 `id10` 、 `id11` 和 `id12` 。
- `ANYSTRING` – 使 `@` 能够匹配任何字符串。你可以将其与 `~` 和 `&` 结合使用以进行排除。示例： `@&.*error.*&.*[0-9]{3}.*` 匹配同时包含“error”一词和三个数字序列的字符串。

## 不支持的特性

Lucene 的引擎不支持以下常用的正则表达式方式：

- `^` – 行首
- `$` – 行尾

## 示例

要尝试正则表达式，请将以下文档索引到 `logs` 索引中：

```
PUT /logs/_doc/1
{
  "message": "error404"
}

PUT /logs/_doc/2
{
  "message": "error500"
}

PUT /logs/_doc/3
{
  "message": "error1a"
}

```

### 示例：包含正则表达式的简单查询

以下 `regexp` 查询返回 `message` 字段的全部值与“error”后跟一个或多个数字的模式匹配的文档。如果值仅包含该模式作为子字符串，则不匹配：

```
GET /logs/_search
{
  "query": {
    "regexp": {
      "message": {
        "value": "error[0-9]+"
      }
    }
  }
}

```

此查询匹配 `error404` 和 `error500` :

```
{
  "took": 28,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "logs",
        "_id": "1",
        "_score": 1,
        "_source": {
          "message": "error404"
        }
      },
      {
        "_index": "logs",
        "_id": "2",
        "_score": 1,
        "_source": {
          "message": "error500"
        }
      }
    ]
  }
}
```

### 示例：使用可选运算符

以下查询匹配 `message` 字段完全匹配以“error”开头，后跟 400 到 500（含）之间的数字的字符串。 `INTERVAL` 标志启用 `<min-max>` 语法用于数值范围：

```
GET /logs/_search
{
  "query": {
    "regexp": {
      "message": {
        "value": "error<400-500>",
        "flags": "INTERVAL"
      }
    }
  }
}

```

此查询匹配 error404 和 error500 :

```
{
  "took": 22,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "logs",
        "_id": "1",
        "_score": 1,
        "_source": {
          "message": "error404"
        }
      },
      {
        "_index": "logs",
        "_id": "2",
        "_score": 1,
        "_source": {
          "message": "error500"
        }
      }
    ]
  }
}
```

### 示例：使用 ANYSTRING

当 `ANYSTRING` 标志启用时， `@` 运算符匹配整个字符串。这在与交集（ `&` ）结合使用时很有用，因为它允许你在特定条件下构建匹配完整字符串的查询。

以下查询匹配包含单词“error”和三个数字序列的消息。使用 `ANYSTRING` 来断言整个字段必须匹配两个模式的交集：

```
GET /logs/_search
{
  "query": {
    "regexp": {
      "message.keyword": {
        "value": "@&.*error.*&.*[0-9]{3}.*",
        "flags": "ANYSTRING|INTERSECTION"
      }
    }
  }
}

```

此查询匹配 error404 和 error500 :

```
{
  "took": 20,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "logs",
        "_id": "1",
        "_score": 1,
        "_source": {
          "message": "error404"
        }
      },
      {
        "_index": "logs",
        "_id": "2",
        "_score": 1,
        "_source": {
          "message": "error500"
        }
      }
    ]
  }
}
```

请注意，此查询也将匹配 xerror500 、 error500x 和 errorxx500 。


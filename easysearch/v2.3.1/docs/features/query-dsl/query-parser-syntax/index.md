---
title: "查询字符串语法"
date: 0001-01-01
description: "Lucene 查询解析器语法：词项、字段、通配符、模糊、邻近、范围、布尔操作符、分组与转义。"
summary: "查询字符串语法 #  Easysearch 的 query_string 和 simple_query_string 查询底层使用 Lucene 查询解析器（Query Parser）将查询字符串解析为查询对象。本页详细描述查询解析器支持的完整语法。
相关指南 #    Query String 查询 — query_string 查询的 API 参数  Simple Query String 查询 — 更宽松的查询字符串语法  正则表达式语法 — regexp 查询和 query_string 中使用的正则语法   词项（Terms） #  查询字符串由词项和操作符组成。词项有两种类型：
 单个词项：一个单词，如 test、hello 短语：用双引号包围的一组词，如 &quot;hello dolly&quot;  多个词项可以通过布尔操作符组合成更复杂的查询。
 注意：查询字符串中的词项和短语会经过索引时使用的相同分析器处理。因此，选择不会干扰查询词项的分析器非常重要。
  字段（Fields） #  Easysearch 支持按字段搜索。搜索时可以指定字段名，也可以使用默认字段。
语法为字段名后跟冒号 :，然后是要搜索的词项：
title:&#34;The Right Way&#34; AND content:go 上面的查询在 title 字段中查找短语 &ldquo;The Right Way&rdquo;，在 content 字段中查找词 &ldquo;go&rdquo;。"
---


# 查询字符串语法

Easysearch 的 [`query_string`]({{< relref "/docs/features/fulltext-search/full-text/query-string.md" >}}) 和 [`simple_query_string`]({{< relref "/docs/features/fulltext-search/full-text/simple-query-string.md" >}}) 查询底层使用 Lucene 查询解析器（Query Parser）将查询字符串解析为查询对象。本页详细描述查询解析器支持的完整语法。

## 相关指南

- [Query String 查询]({{< relref "/docs/features/fulltext-search/full-text/query-string.md" >}}) — `query_string` 查询的 API 参数
- [Simple Query String 查询]({{< relref "/docs/features/fulltext-search/full-text/simple-query-string.md" >}}) — 更宽松的查询字符串语法
- [正则表达式语法]({{< relref "/docs/features/query-dsl/regex-syntax.md" >}}) — `regexp` 查询和 `query_string` 中使用的正则语法

---

## 词项（Terms）

查询字符串由**词项**和**操作符**组成。词项有两种类型：

- **单个词项**：一个单词，如 `test`、`hello`
- **短语**：用双引号包围的一组词，如 `"hello dolly"`

多个词项可以通过布尔操作符组合成更复杂的查询。

> **注意**：查询字符串中的词项和短语会经过索引时使用的相同分析器处理。因此，选择不会干扰查询词项的分析器非常重要。

---

## 字段（Fields）

Easysearch 支持按字段搜索。搜索时可以指定字段名，也可以使用默认字段。

语法为字段名后跟冒号 `:`，然后是要搜索的词项：

```
title:"The Right Way" AND content:go
```

上面的查询在 `title` 字段中查找短语 "The Right Way"，在 `content` 字段中查找词 "go"。

> **注意**：字段指示符仅对其直接前面的词项有效。例如：
>
> ```
> title:Do it right
> ```
>
> 只有 "Do" 在 `title` 字段中搜索，"it" 和 "right" 将在默认字段中搜索。

---

## 词项修饰符

### 通配符搜索

支持单字符和多字符通配符：

| 通配符 | 含义 | 示例 |
|--------|------|------|
| `?` | 匹配任意单个字符 | `te?t` → 匹配 `test`、`text` |
| `*` | 匹配零个或多个字符 | `test*` → 匹配 `test`、`tests`、`tester` |

通配符可以出现在词项中间：

```
te*t
```

> **注意**：不能将 `*` 或 `?` 作为搜索词的第一个字符（性能原因）。如果需要左通配符匹配，考虑使用 [通配符字段类型（Wildcard）]({{< relref "/docs/features/mapping-and-analysis/field-types/wildcard.md" >}})。

### 模糊搜索

模糊搜索基于 Damerau-Levenshtein 编辑距离算法，能找到拼写相似的词项。在词项末尾加波浪号 `~` 即可启用：

```
roam~
```

这会匹配 `foam`、`roams` 等拼写相近的词。

可以指定编辑距离参数（0~2），值越大匹配越宽松：

```
roam~1
```

默认编辑距离为 2。

### 邻近搜索

邻近搜索查找彼此在指定距离内的词。在**短语**末尾加 `~` 和距离值：

```
"jakarta apache"~10
```

这会查找 "jakarta" 和 "apache" 在 10 个词距离内出现的文档。

### 范围搜索

范围搜索匹配字段值在指定上下界之间的文档：

| 语法 | 含义 | 示例 |
|------|------|------|
| `[min TO max]` | **包含边界**（闭区间） | `date:[2020-01-01 TO 2020-12-31]` |
| `{min TO max}` | **排除边界**（开区间） | `title:{Aida TO Carmen}` |
| `[min TO max}` | 左闭右开 | `count:[1 TO 100}` |
| `{min TO max]` | 左开右闭 | `count:{0 TO 100]` |

范围搜索不限于日期字段，也适用于数值和字符串字段：

```
age:[18 TO 65]
title:{Alpha TO Omega}
```

使用通配符表示无上限或无下限：

```
age:[18 TO *]
age:{* TO 65]
```

### 权重提升（Boosting）

使用 `^` 符号和提升因子来控制词项的相关性权重：

```
jakarta^4 apache
```

这会使包含 "jakarta" 的文档更相关。短语也可以提升权重：

```
"jakarta apache"^4 "Apache Lucene"
```

默认提升因子为 1。提升因子必须为正数，但可以小于 1（如 `0.2`）。

---

## 布尔操作符

布尔操作符用于组合多个词项的逻辑关系。

> **注意**：布尔操作符关键字必须**全部大写**。

### OR（默认操作符）

`OR` 是默认的连接操作符。如果两个词项之间没有布尔操作符，则使用 OR。可以用 `||` 替代：

```
"jakarta apache" jakarta
"jakarta apache" OR jakarta
```

匹配包含任意一个词项的文档。

### AND

`AND` 操作符要求两个词项都存在于文档中。可以用 `&&` 替代：

```
"jakarta apache" AND "Apache Lucene"
```

### NOT

`NOT` 操作符排除包含指定词项的文档。可以用 `!` 替代：

```
"jakarta apache" NOT "Apache Lucene"
```

> **注意**：`NOT` 不能单独使用。以下查询不会返回任何结果：
> ```
> NOT "jakarta apache"
> ```

### 必须出现（+）

`+` 操作符要求其后的词项必须存在于文档中：

```
+jakarta lucene
```

匹配必须包含 "jakarta"、可选包含 "lucene" 的文档。

### 必须不出现（-）

`-` 操作符排除包含其后词项的文档：

```
"jakarta apache" -"Apache Lucene"
```

### 操作符总结

| 操作符 | 等价符号 | 含义 |
|--------|----------|------|
| `OR` | `\|\|` | 任一词项匹配即可（默认） |
| `AND` | `&&` | 所有词项都必须匹配 |
| `NOT` | `!` | 排除包含该词项的文档 |
| `+` | — | 该词项必须出现 |
| `-` | — | 该词项必须不出现 |

---

## 分组（Grouping）

使用括号 `()` 对子句进行分组，控制布尔逻辑的优先级：

```
(jakarta OR apache) AND website
```

这确保 "website" 必须存在，同时 "jakarta" 或 "apache" 至少存在一个。

### 字段分组

对同一字段的多个子句进行分组：

```
title:(+return +"pink panther")
```

这会在 `title` 字段中查找同时包含 "return" 和 "pink panther" 的文档。

---

## 转义特殊字符

以下字符在查询语法中有特殊含义，如果要按字面值匹配，需要用反斜杠 `\` 转义：

```
+ - && || ! ( ) { } [ ] ^ " ~ * ? : \
```

例如，搜索 `(1+1):2`：

```
\(1\+1\)\:2
```

---

## 在 Easysearch 中使用

### query_string 查询

```json
GET articles/_search
{
  "query": {
    "query_string": {
      "default_field": "content",
      "query": "title:(Easysearch AND 教程) AND date:[2025-01-01 TO 2025-12-31]"
    }
  }
}
```

### simple_query_string 查询

`simple_query_string` 使用简化的语法，不会因语法错误而报错：

| 操作符 | query_string | simple_query_string |
|--------|-------------|---------------------|
| AND | `AND` 或 `&&` | `+` |
| OR | `OR` 或 `\|\|` | `\|` |
| NOT | `NOT` 或 `!` | `-` |
| 短语 | `"..."` | `"..."` |
| 前缀 | `term*` | `term*` |
| 模糊 | `term~N` | `term~N` |
| 邻近 | `"..."~N` | `"..."~N` |
| 分组 | `(...)` | `(...)` |
| 字段指定 | `field:term` | ❌ 不支持 |
| 范围 | `[min TO max]` | ❌ 不支持 |
| 正则 | `/pattern/` | ❌ 不支持 |
| 提升 | `term^N` | ❌ 不支持 |

### 常用查询示例

```
# 多字段搜索
title:Easysearch AND content:"搜索引擎"

# 模糊匹配 + 邻近搜索
serach~1 AND "全文 搜索"~3

# 范围 + 通配符
date:[2025-01-01 TO *] AND author:张*

# 必须/可选/排除
+Easysearch +搜索 -Elasticsearch

# 分组 + 权重提升
(title:搜索引擎^3 OR content:搜索引擎) AND status:published
```

---

## 注意事项

| 注意项 | 说明 |
|--------|------|
| 大小写 | 布尔操作符（`AND`、`OR`、`NOT`）必须全大写，否则被当作普通词项 |
| 分析器 | 查询字符串中的词项会经过字段的分析器处理，确保分析器配置正确 |
| 默认操作符 | 默认使用 `OR`，可以通过 `default_operator` 参数改为 `AND` |
| 错误处理 | `query_string` 对语法错误严格报错，面向终端用户建议使用 `simple_query_string` |
| 性能 | 避免前导通配符（`*keyword`），会触发全量词项扫描 |
| 嵌套文档 | `query_string` 不会返回嵌套文档，需使用 `nested` 查询 |


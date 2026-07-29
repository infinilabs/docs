---
title: "全文搜索"
date: 0001-01-01
summary: "全文搜索 #  Easysearch SQL 提供了一组相关性函数，可以在 WHERE 子句中使用，将 SQL 查询与 Easysearch 强大的全文搜索能力相结合。这些函数在底层会翻译为对应的 Easysearch DSL 查询。
函数一览 #     函数 等价函数 底层 DSL 用途     MATCH_QUERY MATCHQUERY match 单字段全文匹配   MULTI_MATCH MULTIMATCH、MULTIMATCHQUERY multi_match 多字段全文匹配   MATCH_PHRASE MATCHPHRASE、MATCHPHRASEQUERY match_phrase 短语匹配   QUERY — query_string Lucene 查询字符串   SCORE SCOREQUERY、SCORE_QUERY constant_score 相关性评分包装     MATCH_QUERY #  在单个文本字段上执行全文匹配查询。
语法 #  MATCH_QUERY(field, text) -- 或 field = MATCH_QUERY(text)    参数 说明     field 要搜索的字段名（必须是 text 类型）   text 搜索文本    示例 #  POST /_sql { &#34;query&#34;: &#34;SELECT account_number, address FROM accounts WHERE MATCH_QUERY(address, &#39;Holmes&#39;)&#34; } 等价 DSL："
---


# 全文搜索

Easysearch SQL 提供了一组相关性函数，可以在 `WHERE` 子句中使用，将 SQL 查询与 Easysearch 强大的全文搜索能力相结合。这些函数在底层会翻译为对应的 Easysearch DSL 查询。

## 函数一览

| 函数 | 等价函数 | 底层 DSL | 用途 |
|------|---------|---------|------|
| `MATCH_QUERY` | `MATCHQUERY` | `match` | 单字段全文匹配 |
| `MULTI_MATCH` | `MULTIMATCH`、`MULTIMATCHQUERY` | `multi_match` | 多字段全文匹配 |
| `MATCH_PHRASE` | `MATCHPHRASE`、`MATCHPHRASEQUERY` | `match_phrase` | 短语匹配 |
| `QUERY` | — | `query_string` | Lucene 查询字符串 |
| `SCORE` | `SCOREQUERY`、`SCORE_QUERY` | `constant_score` | 相关性评分包装 |

---

## MATCH_QUERY

在单个文本字段上执行全文匹配查询。

### 语法

```sql
MATCH_QUERY(field, text)
-- 或
field = MATCH_QUERY(text)
```

| 参数 | 说明 |
|------|------|
| `field` | 要搜索的字段名（必须是 `text` 类型） |
| `text` | 搜索文本 |

### 示例

```json
POST /_sql
{
  "query": "SELECT account_number, address FROM accounts WHERE MATCH_QUERY(address, 'Holmes')"
}
```

等价 DSL：

```json
{"query": {"match": {"address": {"query": "Holmes"}}}}
```

结果：

| account_number | address |
|---|---|
| 1 | 880 Holmes Lane |

也可以使用等号写法：

```sql
SELECT account_number, address FROM accounts WHERE address = MATCH_QUERY('Holmes')
```

---

## MULTI_MATCH

在多个字段上同时执行全文匹配查询，支持通配符字段模式。

### 语法

```sql
MULTI_MATCH('query'='text', 'fields'='field_pattern')
```

| 参数 | 说明 | 示例 |
|------|------|------|
| `query` | 搜索文本 | `'Dale'` |
| `fields` | 字段名或通配符模式 | `'*name'`、`'firstname,lastname'` |

### 示例

搜索 firstname 或 lastname 中包含 "Dale" 的文档：

```json
POST /_sql
{
  "query": "SELECT firstname, lastname FROM accounts WHERE MULTI_MATCH('query'='Dale', 'fields'='*name')"
}
```

等价 DSL：

```json
{"query": {"multi_match": {"query": "Dale", "fields": ["*name^1.0"], "type": "best_fields"}}}
```

结果：

| firstname | lastname |
|---|---|
| Dale | Adams |

---

## MATCH_PHRASE

匹配精确的短语序列，词项必须按顺序连续出现。

### 语法

```sql
MATCH_PHRASE(field, phrase)
```

| 参数 | 说明 |
|------|------|
| `field` | 要搜索的字段名 |
| `phrase` | 精确短语文本 |

### 示例

```json
POST /_sql
{
  "query": "SELECT account_number, address FROM accounts WHERE MATCH_PHRASE(address, '880 Holmes Lane')"
}
```

等价 DSL：

```json
{"query": {"match_phrase": {"address": {"query": "880 Holmes Lane"}}}}
```

结果：

| account_number | address |
|---|---|
| 1 | 880 Holmes Lane |

---

## QUERY

使用 Lucene 查询字符串语法执行搜索，支持布尔运算、通配符、正则表达式和邻近搜索等高级查询语法。

### 语法

```sql
QUERY('query_string')
```

查询字符串遵循 [Lucene 查询语法](https://lucene.apache.org/core/9_0_0/queryparser/org/apache/lucene/queryparser/classic/package-summary.html)，常用写法包括：

| 写法 | 说明 |
|------|------|
| `field:value` | 指定字段搜索 |
| `AND` / `OR` / `NOT` | 布尔运算 |
| `field:val*` | 通配符 |
| `"exact phrase"` | 短语搜索 |
| `field:[min TO max]` | 范围查询 |

> 如果查询字符串包含语法错误，将抛出解析异常。

### 示例

```json
POST /_sql
{
  "query": "SELECT account_number, address FROM accounts WHERE QUERY('address:Lane OR address:Street')"
}
```

等价 DSL：

```json
{"query": {"query_string": {"query": "address:Lane OR address:Street"}}}
```

结果：

| account_number | address |
|---|---|
| 1 | 880 Holmes Lane |
| 6 | 671 Bristol Street |
| 13 | 789 Madison Street |

---

## SCORE

将全文查询包装为评分查询，使每条匹配文档带有相关性得分。使用 `_score` 隐式变量可以在 SELECT 或 ORDER BY 中访问该分数。

### 语法

```sql
SCORE(match_expression [, boost])
```

| 参数 | 说明 | 默认值 |
|------|------|-------|
| `match_expression` | 全文匹配表达式（如 `MATCH_QUERY(...)` 等） | — |
| `boost` | 可选的浮点权重因子 | `1.0` |

### 示例

使用不同的 boost 权重区分不同匹配条件的优先级：

```json
POST /_sql
{
  "query": "SELECT account_number, address, _score FROM accounts WHERE SCORE(MATCH_QUERY(address, 'Lane'), 0.5) OR SCORE(MATCH_QUERY(address, 'Street'), 100) ORDER BY _score"
}
```

结果：

| account_number | address | _score |
|---|---|---|
| 1 | 880 Holmes Lane | 0.5 |
| 6 | 671 Bristol Street | 100.0 |
| 13 | 789 Madison Street | 100.0 |

> `_score` 字段只有在使用 `SCORE` 函数后才会有非零值。普通的 `MATCH_QUERY` 在 `WHERE` 中作为过滤条件使用，不产生评分。

---

## 组合使用

全文搜索函数可以与普通的 SQL 条件组合使用：

```sql
-- 全文搜索 + 精确过滤
SELECT firstname, lastname, address, _score
FROM accounts
WHERE SCORE(MATCH_QUERY(address, 'Street'), 10)
  AND age > 30
ORDER BY _score DESC

-- 全文搜索 + 聚合
SELECT city, COUNT(*) AS cnt
FROM accounts
WHERE MATCH_QUERY(address, 'Street')
GROUP BY city
ORDER BY cnt DESC
```

## 相关链接

- [SQL 查询总览]({{< relref "/docs/features/sql/" >}})
- [全文搜索功能]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})


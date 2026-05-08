---
title: "基础查询"
date: 0001-01-01
summary: "基础查询 #  概述 #  SELECT 语句是从 Easysearch 索引中检索数据的核心语法。完整语法如下：
SELECT [ALL | DISTINCT] (* | expression) [[AS] alias] [, ...] FROM index_name [WHERE predicates] [GROUP BY expression [, ...] [HAVING predicates]] [ORDER BY expression [ASC | DESC] [NULLS {FIRST | LAST}] [, ...]] [LIMIT [offset, ] size]  语句末尾允许加分号 ;，这对 BI 工具和 Excel 生成的查询很有用。
 执行顺序 #  SQL 各子句的实际执行顺序与书写顺序不同：
FROM index -- ① 确定数据源 → WHERE -- ② 行级过滤 → GROUP BY -- ③ 分组 → HAVING -- ④ 组级过滤 → SELECT -- ⑤ 字段投影 → ORDER BY -- ⑥ 排序 → LIMIT -- ⑦ 分页  SELECT #  SELECT 子句指定要检索的字段。"
---


# 基础查询

## 概述

SELECT 语句是从 Easysearch 索引中检索数据的核心语法。完整语法如下：

```sql
SELECT [ALL | DISTINCT] (* | expression) [[AS] alias] [, ...]
FROM index_name
[WHERE predicates]
[GROUP BY expression [, ...]
 [HAVING predicates]]
[ORDER BY expression [ASC | DESC] [NULLS {FIRST | LAST}] [, ...]]
[LIMIT [offset, ] size]
```

> 语句末尾允许加分号 `;`，这对 BI 工具和 Excel 生成的查询很有用。

### 执行顺序

SQL 各子句的实际执行顺序与书写顺序不同：

```
FROM index         -- ① 确定数据源
 → WHERE           -- ② 行级过滤
  → GROUP BY       -- ③ 分组
   → HAVING        -- ④ 组级过滤
    → SELECT       -- ⑤ 字段投影
     → ORDER BY    -- ⑥ 排序
      → LIMIT      -- ⑦ 分页
```

---

## SELECT

SELECT 子句指定要检索的字段。

### 查询所有字段

```json
POST /_sql
{
  "query": "SELECT * FROM accounts"
}
```

### 查询指定字段

```json
POST /_sql
{
  "query": "SELECT firstname, lastname, balance FROM accounts"
}
```

### 字段别名

使用 `AS` 为字段设置别名，提高可读性：

```json
POST /_sql
{
  "query": "SELECT account_number AS num, balance AS bal FROM accounts"
}
```

### DISTINCT 去重

返回字段的唯一值（底层翻译为 terms 聚合）：

```json
POST /_sql
{
  "query": "SELECT DISTINCT age FROM accounts"
}
```

DISTINCT 也可以与表达式结合使用：

```sql
SELECT DISTINCT SUBSTRING(lastname, 1, 1) FROM accounts
```

---

## FROM

FROM 子句指定查询的数据源（索引）。

### 索引别名

```sql
SELECT acc.firstname, acc.lastname FROM accounts acc
```

### 索引模式匹配

通过通配符从多个名称相似的索引查询数据，适合时间序列索引（如 `logs-2025.02.*`）：

```sql
SELECT * FROM account*
```

### 子查询

FROM 子句支持子查询（详见[复杂查询]({{< relref "complex" >}})）。

---

## WHERE

WHERE 子句对文档进行行级过滤。支持以下谓词：

| 运算符 | 说明 | 示例 |
|-------|------|------|
| `=`, `<>`, `!=` | 相等/不等 | `WHERE age = 30` |
| `>`, `>=`, `<`, `<=` | 比较 | `WHERE balance >= 10000` |
| `AND`, `OR`, `NOT` | 逻辑组合 | `WHERE age > 30 AND gender = 'M'` |
| `IN` | 集合匹配 | `WHERE state IN ('CA', 'NY', 'TX')` |
| `BETWEEN ... AND` | 范围 | `WHERE age BETWEEN 25 AND 35` |
| `LIKE` | 模式匹配 | `WHERE lastname LIKE 'D%'` |
| `IS NULL` / `IS NOT NULL` | 空值检测 | `WHERE employer IS NULL` |

### 基本过滤

```json
POST /_sql
{
  "query": "SELECT account_number, balance FROM accounts WHERE balance > 10000"
}
```

翻译为 DSL：

```json
{
  "query": {
    "bool": {
      "filter": [{
        "range": {"balance": {"gt": 10000}}
      }]
    }
  }
}
```

### 组合条件

```sql
SELECT firstname, age, balance
FROM accounts
WHERE age > 30 AND balance > 5000 AND state IN ('IL', 'TN')
```

### NULL 检测

Easysearch 允许灵活的文档结构，字段可能缺失。使用 `IS NULL` / `IS NOT NULL` 检测：

```sql
SELECT account_number, employer FROM accounts WHERE employer IS NULL
```

> 当前 SQL 插件不区分"字段缺失"和"字段值为 null"。

---

## GROUP BY

GROUP BY 子句将文档按字段值分组，通常与聚合函数配合使用。详细的聚合语法见[聚合查询]({{< relref "aggregations" >}})。

GROUP BY 表达式支持三种形式：

```sql
-- 1. 按字段名
GROUP BY gender

-- 2. 按 SELECT 列表中的序号
GROUP BY 1

-- 3. 按表达式
GROUP BY ABS(age)
```

### 示例

```sql
SELECT gender, COUNT(*), AVG(age) FROM accounts GROUP BY gender
```

```
+--------+-----------+----------+
| gender | COUNT(*)  | AVG(age) |
+--------+-----------+----------+
| F      | 1         | 28.0     |
| M      | 3         | 33.67    |
+--------+-----------+----------+
```

### 按别名分组

```sql
SELECT account_number AS num FROM accounts GROUP BY num
```

### 按标量函数分组

```sql
SELECT ABS(age) AS a, COUNT(*) FROM accounts GROUP BY ABS(age)
```

---

## HAVING

HAVING 子句在 GROUP BY 之后对聚合结果进行过滤。

```sql
SELECT age, MAX(balance) FROM accounts
GROUP BY age
HAVING MIN(balance) > 10000
```

HAVING 中可以使用与 SELECT 不同的聚合函数，也可以引用 SELECT 中的别名：

```sql
SELECT gender, SUM(age) AS s FROM accounts GROUP BY gender HAVING s > 100
```

---

## ORDER BY

ORDER BY 子句指定结果排序方式。

```sql
-- 按字段名排序（ASC 升序为默认）
SELECT * FROM accounts ORDER BY balance DESC

-- 按序号排序
SELECT firstname, age FROM accounts ORDER BY 2 DESC

-- 按别名排序
SELECT balance AS b FROM accounts ORDER BY b

-- 空值排序控制
SELECT * FROM accounts ORDER BY employer NULLS LAST

-- 按聚合函数排序
SELECT gender, MAX(age) FROM accounts GROUP BY gender ORDER BY MAX(age) DESC
```

---

## LIMIT

LIMIT 子句控制返回结果的数量。建议与 ORDER BY 配合使用以获得确定性结果。

```sql
-- 返回前 10 条
SELECT * FROM accounts ORDER BY balance DESC LIMIT 10

-- 跳过 5 条后返回 10 条（分页）
SELECT * FROM accounts ORDER BY account_number LIMIT 5, 10

-- 使用 OFFSET 语法
SELECT * FROM accounts ORDER BY age LIMIT 10 OFFSET 5
```

> 对于大偏移量的分页，性能较低。大规模分页建议使用[游标分页]({{< relref "/docs/features/sql/" >}}#游标分页)。



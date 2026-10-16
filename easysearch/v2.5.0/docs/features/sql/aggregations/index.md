---
title: "聚合查询"
date: 0001-01-01
summary: "聚合查询 #  聚合函数对一组文档进行计算并返回单个值。通常与 GROUP BY 子句组合使用，对数据进行分组后计算汇总统计量。
 聚合函数 #  基本聚合函数 #     函数 说明 示例     COUNT(*) 计算所有行数 SELECT COUNT(*) FROM accounts   COUNT(field) 计算非 NULL 值行数 SELECT COUNT(age) FROM accounts   COUNT(DISTINCT field) 计算字段不同值数量 SELECT COUNT(DISTINCT gender) FROM accounts   SUM(expr) 求和 SELECT SUM(balance) FROM accounts   AVG(expr) 算术平均值 SELECT AVG(age) FROM accounts   MAX(expr) 最大值 SELECT MAX(balance) FROM accounts   MIN(expr) 最小值 SELECT MIN(balance) FROM accounts    统计聚合函数 #     函数 同义函数 说明     VAR_POP(expr) VARIANCE(expr) 总体方差   VAR_SAMP(expr) — 样本方差   STDDEV_POP(expr) STD(expr)、STDDEV(expr) 总体标准差   STDDEV_SAMP(expr) — 样本标准差    使用示例 #  -- 基本聚合 SELECT COUNT(*), SUM(balance), AVG(age), MAX(balance), MIN(balance) FROM accounts -- 分组聚合 SELECT gender, COUNT(*) AS cnt, AVG(age) AS avg_age FROM accounts GROUP BY gender -- 聚合表达式 SELECT gender, SUM(age) * 2 AS double_sum FROM accounts GROUP BY gender -- 表达式作为聚合参数 SELECT gender, SUM(age * 2) AS sum_double FROM accounts GROUP BY gender COUNT 的特殊形式 #     形式 行为     COUNT(field) 仅计数字段值非 NULL 的行   COUNT(*) 计数所有输入行（包含 NULL）   COUNT(1) 等同于 COUNT(*)，任何非 NULL 字面量均可   COUNT(DISTINCT field) 计数字段的不重复值    SELECT COUNT(DISTINCT gender), COUNT(gender) FROM accounts    COUNT(DISTINCT gender) COUNT(gender)     2 4     GROUP BY #  GROUP BY 按表达式对结果分组。支持三种写法："
---


# 聚合查询

聚合函数对一组文档进行计算并返回单个值。通常与 `GROUP BY` 子句组合使用，对数据进行分组后计算汇总统计量。

---

## 聚合函数

### 基本聚合函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `COUNT(*)` | 计算所有行数 | `SELECT COUNT(*) FROM accounts` |
| `COUNT(field)` | 计算非 NULL 值行数 | `SELECT COUNT(age) FROM accounts` |
| `COUNT(DISTINCT field)` | 计算字段不同值数量 | `SELECT COUNT(DISTINCT gender) FROM accounts` |
| `SUM(expr)` | 求和 | `SELECT SUM(balance) FROM accounts` |
| `AVG(expr)` | 算术平均值 | `SELECT AVG(age) FROM accounts` |
| `MAX(expr)` | 最大值 | `SELECT MAX(balance) FROM accounts` |
| `MIN(expr)` | 最小值 | `SELECT MIN(balance) FROM accounts` |

### 统计聚合函数

| 函数 | 同义函数 | 说明 |
|------|---------|------|
| `VAR_POP(expr)` | `VARIANCE(expr)` | 总体方差 |
| `VAR_SAMP(expr)` | — | 样本方差 |
| `STDDEV_POP(expr)` | `STD(expr)`、`STDDEV(expr)` | 总体标准差 |
| `STDDEV_SAMP(expr)` | — | 样本标准差 |

### 使用示例

```sql
-- 基本聚合
SELECT COUNT(*), SUM(balance), AVG(age), MAX(balance), MIN(balance)
FROM accounts

-- 分组聚合
SELECT gender, COUNT(*) AS cnt, AVG(age) AS avg_age
FROM accounts
GROUP BY gender

-- 聚合表达式
SELECT gender, SUM(age) * 2 AS double_sum
FROM accounts
GROUP BY gender

-- 表达式作为聚合参数
SELECT gender, SUM(age * 2) AS sum_double
FROM accounts
GROUP BY gender
```

### COUNT 的特殊形式

| 形式 | 行为 |
|------|------|
| `COUNT(field)` | 仅计数字段值非 NULL 的行 |
| `COUNT(*)` | 计数所有输入行（包含 NULL） |
| `COUNT(1)` | 等同于 `COUNT(*)`，任何非 NULL 字面量均可 |
| `COUNT(DISTINCT field)` | 计数字段的不重复值 |

```sql
SELECT COUNT(DISTINCT gender), COUNT(gender) FROM accounts
```

| COUNT(DISTINCT gender) | COUNT(gender) |
|---|---|
| 2 | 4 |

---

## GROUP BY

GROUP BY 按表达式对结果分组。支持三种写法：

```sql
-- 按字段名
SELECT gender, SUM(age) FROM accounts GROUP BY gender

-- 按 SELECT 列表中的序号
SELECT gender, SUM(age) FROM accounts GROUP BY 1

-- 按表达式
SELECT ABS(account_number), SUM(age) FROM accounts GROUP BY ABS(account_number)
```

---

## HAVING

HAVING 在分组之后对聚合结果进行过滤（WHERE 是在分组之前对行进行过滤）。

### 带 GROUP BY 的 HAVING

```sql
SELECT gender, SUM(age)
FROM accounts
GROUP BY gender
HAVING SUM(age) > 100
```

HAVING 中可以使用 SELECT 别名：

```sql
SELECT gender, SUM(age) AS s
FROM accounts
GROUP BY gender
HAVING s > 100
```

> HAVING 中的聚合函数不要求与 SELECT 中相同。例如 SELECT 使用 `SUM()`，HAVING 可以用 `MIN()` 过滤。

### 不带 GROUP BY 的 HAVING

HAVING 也可以不配合 GROUP BY 使用，适用于对全表聚合结果做条件判断：

```sql
SELECT 'Total of age > 100'
FROM accounts
HAVING SUM(age) > 100
```

---

## FILTER 子句

`FILTER` 子句为聚合函数设置独立的过滤条件，只有满足条件的行才参与该聚合计算。

### 语法

```sql
aggregation_function(expr) FILTER(WHERE condition)
```

### 带 GROUP BY 的 FILTER

每个聚合可以有不同的过滤条件：

```sql
SELECT
  AVG(age) FILTER(WHERE balance > 10000) AS high_bal_avg,
  AVG(age) FILTER(WHERE balance <= 10000) AS low_bal_avg,
  gender
FROM accounts
GROUP BY gender
```

### 不带 GROUP BY 的 FILTER

```sql
SELECT
  COUNT(*) AS total,
  COUNT(*) FILTER(WHERE age > 34) AS over_34
FROM accounts
```

| total | over_34 |
|---|---|
| 4 | 1 |

FILTER 也可以与 DISTINCT COUNT 结合：

```sql
SELECT COUNT(DISTINCT firstname) FILTER(WHERE age > 30) AS cnt FROM accounts
```

---

## 窗口函数（Window Functions）

窗口函数在一组相关行（"窗口"）上执行计算，每行保留独立结果，不折叠为单行。

### 语法

```sql
function_name(...) OVER (
  [PARTITION BY expr [, ...]]
  [ORDER BY expr [ASC | DESC] [, ...]]
)
```

### 排名函数

| 函数 | 说明 |
|------|------|
| `ROW_NUMBER()` | 窗口内连续行号，从 1 开始 |
| `RANK()` | 排名，相同值并列，后续排名跳过 |
| `DENSE_RANK()` | 排名，相同值并列，后续排名不跳过 |

### 示例

```sql
-- 按 gender 分区，按 age 降序排名
SELECT firstname, gender, age,
  ROW_NUMBER() OVER (PARTITION BY gender ORDER BY age DESC) AS row_num,
  RANK()       OVER (PARTITION BY gender ORDER BY age DESC) AS rnk,
  DENSE_RANK() OVER (PARTITION BY gender ORDER BY age DESC) AS dense_rnk
FROM accounts

-- 聚合函数也可以作为窗口函数使用
SELECT firstname, age,
  SUM(age) OVER (ORDER BY age) AS running_sum,
  AVG(age) OVER () AS global_avg
FROM accounts
```

---

## 相关链接

- [SQL 查询总览]({{< relref "/docs/features/sql/" >}})
- [聚合分析基础教程]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [内置函数参考]({{< relref "functions" >}})



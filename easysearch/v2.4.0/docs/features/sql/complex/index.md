---
title: "复杂查询"
date: 0001-01-01
summary: "复杂查询 #  本章介绍子查询、JOIN 连接和条件表达式等进阶 SQL 功能。
 子查询（Subquery） #  子查询是嵌套在另一个 SQL 语句中的完整 SELECT 语句，用括号括起来。Easysearch SQL 支持两种形式的子查询。
WHERE IN 子查询 #  在 WHERE 子句中使用 IN 加子查询来过滤符合条件的文档。底层会被转换为等效的 JOIN 查询执行。
POST /_sql { &#34;query&#34;: &#34;&#34;&#34; SELECT a1.firstname, a1.lastname, a1.balance FROM accounts a1 WHERE a1.account_number IN ( SELECT a2.account_number FROM accounts a2 WHERE a2.balance &gt; 10000 ) &#34;&#34;&#34; } 结果：
   a1.firstname a1.lastname a1.balance     Amber Duke 39225   Nanette Bates 32838    FROM 子查询 #  在 FROM 子句中使用子查询作为派生表，子查询必须指定别名。"
---


# 复杂查询

本章介绍子查询、JOIN 连接和条件表达式等进阶 SQL 功能。

---

## 子查询（Subquery）

子查询是嵌套在另一个 SQL 语句中的完整 `SELECT` 语句，用括号括起来。Easysearch SQL 支持两种形式的子查询。

### WHERE IN 子查询

在 `WHERE` 子句中使用 `IN` 加子查询来过滤符合条件的文档。底层会被转换为等效的 JOIN 查询执行。

```json
POST /_sql
{
  "query": """
    SELECT a1.firstname, a1.lastname, a1.balance
    FROM accounts a1
    WHERE a1.account_number IN (
      SELECT a2.account_number
      FROM accounts a2
      WHERE a2.balance > 10000
    )
  """
}
```

结果：

| a1.firstname | a1.lastname | a1.balance |
|---|---|---|
| Amber | Duke | 39225 |
| Nanette | Bates | 32838 |

### FROM 子查询

在 `FROM` 子句中使用子查询作为派生表，子查询必须指定别名。

```json
POST /_sql
{
  "query": """
    SELECT a.f, a.l, a.a
    FROM (
      SELECT firstname AS f, lastname AS l, age AS a
      FROM accounts
      WHERE age > 30
    ) AS a
  """
}
```

结果：

| f | l | a |
|---|---|---|
| Amber | Duke | 32 |
| Dale | Adams | 33 |
| Hattie | Bond | 36 |

FROM 子查询中可以包含聚合和 GROUP BY：

```sql
SELECT avg_balance FROM (
  SELECT AVG(balance) AS avg_balance FROM accounts GROUP BY gender, age
) AS a
```

也支持多层嵌套子查询：

```sql
SELECT name FROM (
  SELECT lastname AS name, age FROM (
    SELECT * FROM accounts WHERE gender = 'M'
  ) AS accounts WHERE age < 35
) AS accounts
```

---

## JOIN

`JOIN` 子句将两个索引的数据按关联条件进行组合。

语法结构参考：

![tableSource](/img/sql/tableSource.png)

![joinPart](/img/sql/joinPart.png)

### INNER JOIN

内连接返回两个索引中满足连接条件的匹配文档。`INNER` 关键字可省略。

```json
POST /_sql
{
  "query": """
    SELECT
      a.account_number, a.firstname, a.lastname,
      e.id, e.name
    FROM accounts a
    JOIN employees_nested e
      ON a.account_number = e.id
  """
}
```

结果：

| a.account_number | a.firstname | a.lastname | e.id | e.name |
|---|---|---|---|---|
| 6 | Hattie | Bond | 6 | Jane Smith |

> 底层使用 Block Hash Join 算法：先基于第一个查询的结果构建哈希表，然后逐行探测第二个查询的结果进行匹配。

### CROSS JOIN

交叉连接（笛卡尔积）返回两个索引所有文档的组合，**不使用 `ON` 子句**。

```json
POST /_sql
{
  "query": """
    SELECT
      a.account_number, a.firstname, a.lastname,
      e.id, e.name
    FROM accounts a
    JOIN employees_nested e
  """
}
```

结果（4 × 3 = 12 行）：

| a.account_number | a.firstname | a.lastname | e.id | e.name |
|---|---|---|---|---|
| 1 | Amber | Duke | 3 | Bob Smith |
| 1 | Amber | Duke | 4 | Susan Smith |
| 1 | Amber | Duke | 6 | Jane Smith |
| 6 | Hattie | Bond | 3 | Bob Smith |
| ... | ... | ... | ... | ... |

> ⚠️ 交叉连接可能产生大量结果，在中等规模索引上也可能触发熔断器以防止内存溢出。

### LEFT OUTER JOIN

左外连接保留左表（第一个索引）的所有文档，即使右表没有匹配记录也会返回（右表字段填充 `NULL`）。`OUTER` 关键字可省略。

```json
POST /_sql
{
  "query": """
    SELECT
      a.account_number, a.firstname, a.lastname,
      e.id, e.name
    FROM accounts a
    LEFT JOIN employees_nested e
      ON a.account_number = e.id
  """
}
```

结果：

| a.account_number | a.firstname | a.lastname | e.id | e.name |
|---|---|---|---|---|
| 1 | Amber | Duke | NULL | NULL |
| 6 | Hattie | Bond | 6 | Jane Smith |
| 13 | Nanette | Bates | NULL | NULL |
| 18 | Dale | Adams | NULL | NULL |

### JOIN 类型总结

| 类型 | 关键字 | ON 子句 | 说明 |
|------|--------|---------|------|
| 内连接 | `[INNER] JOIN ... ON` | 必需 | 只返回匹配行 |
| 交叉连接 | `JOIN`（无 ON） | 不使用 | 笛卡尔积 |
| 左外连接 | `LEFT [OUTER] JOIN ... ON` | 必需 | 保留左表所有行 |

> 当前不支持 `RIGHT JOIN` 和 `FULL OUTER JOIN`。

---

## CASE 条件表达式

`CASE` 表达式用于在 SQL 中实现条件逻辑，类似于编程语言中的 if-else。

### 搜索 CASE

```sql
SELECT account_number,
  CASE
    WHEN balance > 30000 THEN 'high'
    WHEN balance > 10000 THEN 'medium'
    ELSE 'low'
  END AS level
FROM accounts
```

### 简单 CASE

```sql
SELECT firstname,
  CASE gender
    WHEN 'M' THEN 'Male'
    WHEN 'F' THEN 'Female'
    ELSE 'Unknown'
  END AS gender_label
FROM accounts
```

---

## 相关链接

- [SQL 查询总览]({{< relref "/docs/features/sql/" >}})
- [基础查询语法]({{< relref "basics" >}})


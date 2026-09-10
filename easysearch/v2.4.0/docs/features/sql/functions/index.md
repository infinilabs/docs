---
title: "内置函数"
date: 0001-01-01
summary: "内置函数 #  Easysearch SQL 提供 80 多个内置函数，涵盖数学运算、字符串处理、日期时间、条件判断和类型转换等类别。
 数学函数 #     函数 说明 示例     ABS(expr) 绝对值 ABS(-5) → 5   CEIL(expr) / CEILING(expr) 向上取整 CEIL(2.3) → 3   FLOOR(expr) 向下取整 FLOOR(2.7) → 2   ROUND(expr [, d]) 四舍五入到 d 位小数 ROUND(3.1415, 2) → 3.14   TRUNCATE(expr, d) 截断到 d 位小数 TRUNCATE(3.1415, 2) → 3.14   SQRT(expr) 平方根 SQRT(16) → 4."
---


# 内置函数

Easysearch SQL 提供 80 多个内置函数，涵盖数学运算、字符串处理、日期时间、条件判断和类型转换等类别。

---

## 数学函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `ABS(expr)` | 绝对值 | `ABS(-5)` → `5` |
| `CEIL(expr)` / `CEILING(expr)` | 向上取整 | `CEIL(2.3)` → `3` |
| `FLOOR(expr)` | 向下取整 | `FLOOR(2.7)` → `2` |
| `ROUND(expr [, d])` | 四舍五入到 d 位小数 | `ROUND(3.1415, 2)` → `3.14` |
| `TRUNCATE(expr, d)` | 截断到 d 位小数 | `TRUNCATE(3.1415, 2)` → `3.14` |
| `SQRT(expr)` | 平方根 | `SQRT(16)` → `4.0` |
| `CBRT(expr)` | 立方根 | `CBRT(27)` → `3.0` |
| `POWER(base, exp)` / `POW(base, exp)` | 幂运算 | `POWER(2, 10)` → `1024` |
| `EXP(expr)` | e 的 expr 次幂 | `EXP(1)` → `2.7183` |
| `LN(expr)` | 自然对数 | `LN(E())` → `1.0` |
| `LOG(expr)` | 自然对数（同 `LN`） | `LOG(10)` → `2.3026` |
| `LOG2(expr)` | 以 2 为底的对数 | `LOG2(8)` → `3.0` |
| `LOG10(expr)` | 以 10 为底的对数 | `LOG10(100)` → `2.0` |
| `MOD(a, b)` / `MODULUS(a, b)` | 取模 | `MOD(10, 3)` → `1` |
| `SIGN(expr)` | 符号（-1/0/1） | `SIGN(-5)` → `-1` |
| `PI()` | 圆周率 π | `PI()` → `3.14159` |
| `E()` | 自然常数 e | `E()` → `2.71828` |
| `RAND()` / `RANDOM()` | [0, 1) 随机数 | `RAND()` → `0.7523` |
| `CONV(expr, from_base, to_base)` | 进制转换 | `CONV('a', 16, 10)` → `'10'` |
| `CRC32(expr)` | CRC32 校验和 | `CRC32('easysearch')` |

### 三角函数

| 函数 | 说明 |
|------|------|
| `ACOS(expr)` | 反余弦 |
| `ASIN(expr)` | 反正弦 |
| `ATAN(expr)` | 反正切 |
| `ATAN2(y, x)` | 两参数反正切 |
| `COS(expr)` | 余弦 |
| `COT(expr)` | 余切 |
| `SIN(expr)` | 正弦 |
| `TAN(expr)` | 正切 |
| `DEGREES(expr)` | 弧度转角度 |
| `RADIANS(expr)` | 角度转弧度 |

### 示例

```sql
SELECT ABS(balance - 10000) AS diff,
       ROUND(balance / 1000.0, 1) AS balance_k,
       SQRT(age) AS sqrt_age
FROM accounts
```

---

## 字符串函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `LENGTH(str)` | 字符串长度 | `LENGTH('hello')` → `5` |
| `UPPER(str)` | 转大写 | `UPPER('hello')` → `'HELLO'` |
| `LOWER(str)` | 转小写 | `LOWER('HELLO')` → `'hello'` |
| `SUBSTRING(str, pos [, len])` / `SUBSTR(str, pos [, len])` | 截取子串（pos 从 1 开始） | `SUBSTRING('hello', 2, 3)` → `'ell'` |
| `TRIM(str)` | 去除首尾空白 | `TRIM('  hi  ')` → `'hi'` |
| `LTRIM(str)` | 去除左侧空白 | `LTRIM('  hi')` → `'hi'` |
| `RTRIM(str)` | 去除右侧空白 | `RTRIM('hi  ')` → `'hi'` |
| `CONCAT(str1, str2, ...)` | 连接字符串 | `CONCAT('a', 'b', 'c')` → `'abc'` |
| `CONCAT_WS(sep, str1, str2, ...)` | 用分隔符连接 | `CONCAT_WS('-', 'a', 'b')` → `'a-b'` |
| `REPLACE(str, from, to)` | 替换子串 | `REPLACE('hello', 'l', 'r')` → `'herro'` |
| `LOCATE(substr, str [, pos])` | 查找子串位置（从 1 开始） | `LOCATE('ll', 'hello')` → `3` |
| `LEFT(str, len)` | 从左取 len 个字符 | `LEFT('hello', 2)` → `'he'` |
| `RIGHT(str, len)` | 从右取 len 个字符 | `RIGHT('hello', 2)` → `'lo'` |
| `ASCII(str)` | 首字符的 ASCII 码 | `ASCII('A')` → `65` |
| `REVERSE(str)` | 反转字符串 | `REVERSE('hello')` → `'olleh'` |

### 示例

```sql
SELECT UPPER(firstname) AS name,
       LENGTH(address) AS addr_len,
       SUBSTRING(city, 1, 3) AS city_prefix
FROM accounts
WHERE LENGTH(lastname) > 4
```

---

## 日期时间函数

### 构造函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `DATE(expr)` | 提取日期部分 | `DATE('2025-01-15 10:30:00')` → `'2025-01-15'` |
| `TIME(expr)` | 提取时间部分 | `TIME('2025-01-15 10:30:00')` → `'10:30:00'` |
| `TIMESTAMP(expr)` | 转为时间戳 | `TIMESTAMP('2025-01-15')` |
| `MAKETIME(h, m, s)` | 构造时间值 | `MAKETIME(10, 30, 0)` → `'10:30:00'` |

### 提取函数

| 函数 | 同义函数 | 说明 |
|------|---------|------|
| `YEAR(date)` | — | 提取年份 |
| `MONTH(date)` | — | 提取月份（1-12） |
| `MONTHNAME(date)` | — | 月份名称（英文） |
| `DAY(date)` | `DAYOFMONTH(date)` | 提取日（1-31） |
| `DAYNAME(date)` | — | 星期名称（英文） |
| `DAYOFWEEK(date)` | — | 星期几（1=周日，7=周六） |
| `DAYOFYEAR(date)` | — | 一年中第几天（1-366） |
| `WEEK(date)` | — | 一年中第几周 |
| `HOUR(time)` | — | 提取小时 |
| `MINUTE(time)` | — | 提取分钟 |
| `SECOND(time)` | — | 提取秒 |

### 运算函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `ADDDATE(date, INTERVAL n unit)` / `DATE_ADD(date, INTERVAL n unit)` | 日期加法 | `ADDDATE('2025-01-01', INTERVAL 7 DAY)` |
| `SUBDATE(date, INTERVAL n unit)` / `DATE_SUB(date, INTERVAL n unit)` | 日期减法 | `SUBDATE('2025-01-08', INTERVAL 7 DAY)` |
| `FROM_DAYS(n)` | 从天数转日期 | `FROM_DAYS(738886)` |
| `UNIX_TIMESTAMP(date)` | 转 Unix 时间戳 | `UNIX_TIMESTAMP('2025-01-15')` |

INTERVAL 支持的单位：`SECOND`、`MINUTE`、`HOUR`、`DAY`、`WEEK`、`MONTH`、`QUARTER`、`YEAR`。

### 格式化函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `DATE_FORMAT(date, format)` | 按格式字符串输出 | `DATE_FORMAT('2025-01-15', '%Y/%m/%d')` → `'2025/01/15'` |

常用格式占位符：

| 占位符 | 说明 | 示例值 |
|--------|------|--------|
| `%Y` | 四位年份 | `2025` |
| `%m` | 两位月份 | `01` |
| `%d` | 两位日 | `15` |
| `%H` | 24 小时制小时 | `10` |
| `%i` | 分钟 | `30` |
| `%s` | 秒 | `00` |
| `%W` | 星期名称 | `Wednesday` |
| `%M` | 月份名称 | `January` |

### 示例

```sql
SELECT YEAR(hire_date) AS yr,
       MONTH(hire_date) AS mon,
       DAYNAME(hire_date) AS dow,
       DATE_FORMAT(hire_date, '%Y-%m') AS ym
FROM employees
WHERE hire_date > DATE_SUB(NOW(), INTERVAL 1 YEAR)
```

---

## 条件函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `IF(cond, val1, val2)` | 条件为真返回 val1，否则 val2 | `IF(age > 30, 'senior', 'junior')` |
| `IFNULL(expr, val)` | expr 为 NULL 时返回 val | `IFNULL(employer, 'N/A')` |
| `NULLIF(expr1, expr2)` | 两值相等返回 NULL，否则返回 expr1 | `NULLIF(state, 'IL')` |
| `ISNULL(expr)` | 判断是否为 NULL（返回 1 或 0） | `ISNULL(employer)` |

### 与 CASE 表达式的对比

`IF` 函数适合简单的二选一判断，复杂的多分支条件使用 `CASE` 表达式更清晰（参见[复杂查询]({{< relref "complex" >}})）。

```sql
-- IF 函数
SELECT IF(balance > 10000, 'high', 'low') AS level FROM accounts

-- 等效 CASE 表达式
SELECT CASE WHEN balance > 10000 THEN 'high' ELSE 'low' END AS level FROM accounts
```

---

## 类型转换

使用 `CAST` 函数在 SQL 数据类型之间进行转换。

### 语法

```sql
CAST(expression AS target_type)
```

### 支持的目标类型

| 目标类型 | 说明 |
|---------|------|
| `BOOLEAN` | 布尔值 |
| `BYTE` | 8 位有符号整数 |
| `SHORT` | 16 位有符号整数 |
| `INT` / `INTEGER` | 32 位有符号整数 |
| `LONG` | 64 位有符号整数 |
| `FLOAT` | 单精度浮点 |
| `DOUBLE` | 双精度浮点 |
| `STRING` | 字符串 |
| `DATE` | 日期 |
| `TIME` | 时间 |
| `DATETIME` / `TIMESTAMP` | 日期时间 |

### 示例

```sql
SELECT CAST(age AS DOUBLE) AS age_d,
       CAST(account_number AS STRING) AS num_s,
       CAST('2025-01-15' AS DATE) AS dt
FROM accounts
```

---

## 相关链接

- [SQL 查询总览]({{< relref "/docs/features/sql/" >}})
- [聚合查询]({{< relref "aggregations" >}})
- [全文搜索]({{< relref "fulltext" >}})


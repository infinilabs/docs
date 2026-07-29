---
title: "SQL 查询"
date: 0001-01-01
description: "通过标准 SQL 语法查询 Easysearch，支持 REST API、JDBC 驱动和多种输出格式。"
summary: "SQL 查询 #  Easysearch SQL 插件让熟悉关系型数据库的用户无需学习 Query DSL 即可查询数据。插件将 SQL 语句翻译为原生 Easysearch DSL 执行，支持丰富的 SQL 语法——包括聚合、JOIN、子查询、全文搜索函数、窗口函数等。
快速开始 #  POST /_sql { &#34;query&#34;: &#34;SELECT firstname, lastname, balance FROM accounts WHERE balance &gt; 10000 ORDER BY balance DESC LIMIT 10&#34; } 响应（JDBC 格式，默认）：
{ &#34;schema&#34;: [ {&#34;name&#34;: &#34;firstname&#34;, &#34;type&#34;: &#34;text&#34;}, {&#34;name&#34;: &#34;lastname&#34;, &#34;type&#34;: &#34;text&#34;}, {&#34;name&#34;: &#34;balance&#34;, &#34;type&#34;: &#34;long&#34;} ], &#34;datarows&#34;: [ [&#34;Amber&#34;, &#34;Duke&#34;, 39225], [&#34;Nanette&#34;, &#34;Bates&#34;, 32838] ], &#34;total&#34;: 2, &#34;size&#34;: 2, &#34;status&#34;: 200 } REST API #  查询端点 #     方法 路径 说明     POST /_sql 执行 SQL 查询   POST /_sql/_explain 将 SQL 翻译为 Easysearch DSL（不执行）   POST /_sql/_cursor/close 关闭游标/Scroll 上下文    请求体 #  POST /_sql { &#34;query&#34;: &#34;SELECT * FROM my_index WHERE age &gt; 30&#34;, &#34;fetch_size&#34;: 50 }    参数 类型 说明     query String SQL 查询语句   fetch_size Integer 游标分页大小。为 0 或不传时返回全部结果；大于 0 时启用游标分页   cursor String 游标 ID，用于获取后续分页结果   parameters Array 预编译参数（预留）    查询参数 #  通过 URL 参数控制输出格式和行为："
---


# SQL 查询

Easysearch SQL 插件让熟悉关系型数据库的用户无需学习 Query DSL 即可查询数据。插件将 SQL 语句翻译为原生 Easysearch DSL 执行，支持丰富的 SQL 语法——包括聚合、JOIN、子查询、全文搜索函数、窗口函数等。

## 快速开始

```json
POST /_sql
{
  "query": "SELECT firstname, lastname, balance FROM accounts WHERE balance > 10000 ORDER BY balance DESC LIMIT 10"
}
```

响应（JDBC 格式，默认）：

```json
{
  "schema": [
    {"name": "firstname", "type": "text"},
    {"name": "lastname", "type": "text"},
    {"name": "balance", "type": "long"}
  ],
  "datarows": [
    ["Amber", "Duke", 39225],
    ["Nanette", "Bates", 32838]
  ],
  "total": 2,
  "size": 2,
  "status": 200
}
```

## REST API

### 查询端点

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/_sql` | 执行 SQL 查询 |
| POST | `/_sql/_explain` | 将 SQL 翻译为 Easysearch DSL（不执行） |
| POST | `/_sql/_cursor/close` | 关闭游标/Scroll 上下文 |

### 请求体

```json
POST /_sql
{
  "query": "SELECT * FROM my_index WHERE age > 30",
  "fetch_size": 50
}
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `query` | String | SQL 查询语句 |
| `fetch_size` | Integer | 游标分页大小。为 0 或不传时返回全部结果；大于 0 时启用游标分页 |
| `cursor` | String | 游标 ID，用于获取后续分页结果 |
| `parameters` | Array | 预编译参数（预留） |

### 查询参数

通过 URL 参数控制输出格式和行为：

| 参数 | 说明 |
|------|------|
| `format` | 输出格式：`jdbc`（默认）、`json`、`csv`、`raw`/`txt` |
| `pretty` | 美化 JSON 输出 |
| `flat` | 控制字段展平方式 |
| `separator` | CSV 分隔符 |
| `_score` | 是否在结果中包含相关性评分 |
| `_id` | 是否在结果中包含文档 ID |
| `_type` | 是否在结果中包含文档类型 |

### 输出格式

**JDBC 格式**（默认）——结构化 JSON，含 schema 元信息：

```json
POST /_sql
{
  "query": "SELECT firstname, age FROM accounts LIMIT 2"
}
```

```json
{
  "schema": [
    {"name": "firstname", "type": "text"},
    {"name": "age", "type": "integer"}
  ],
  "datarows": [
    ["Amber", 32],
    ["Hattie", 36]
  ],
  "total": 2, "size": 2, "status": 200
}
```

**CSV 格式**：

```
POST /_sql?format=csv
{
  "query": "SELECT firstname, age FROM accounts LIMIT 2"
}
```

```csv
firstname,age
Amber,32
Hattie,36
```

**纯文本表格**（`raw` 或 `txt`）：

```
POST /_sql?format=raw
{
  "query": "SELECT firstname, age FROM accounts LIMIT 2"
}
```

```
firstname | age
----------+----
Amber     | 32
Hattie    | 36
```

### 查询执行计划

使用 `_explain` 端点查看 SQL 如何被翻译为 Easysearch DSL：

```json
POST /_sql/_explain
{
  "query": "SELECT firstname FROM accounts WHERE age > 30"
}
```

```json
{
  "from": 0,
  "size": 200,
  "query": {
    "bool": {
      "filter": [{
        "bool": {
          "must": [{
            "range": {
              "age": {"from": 30, "include_lower": false}
            }
          }]
        }
      }]
    }
  },
  "_source": {
    "includes": ["firstname"]
  }
}
```

### 游标分页

对于大结果集，使用 `fetch_size` 启用游标分页（基于 Scroll API 实现）：

```json
POST /_sql
{
  "query": "SELECT * FROM my_index ORDER BY timestamp",
  "fetch_size": 100
}
```

响应中包含 `cursor` 字段：

```json
{
  "schema": [...],
  "datarows": [...],
  "cursor": "d:eyJhIjp7fSwi..."
}
```

获取下一页：

```json
POST /_sql
{
  "cursor": "d:eyJhIjp7fSwi..."
}
```

当所有数据返回完毕后，响应中不再包含 `cursor` 字段。也可以主动关闭游标释放服务端资源：

```json
POST /_sql/_cursor/close
{
  "cursor": "d:eyJhIjp7fSwi..."
}
```

## 支持的 SQL 语句

| 语句 | 说明 |
|------|------|
| `SELECT` | 数据检索，完整语法支持（WHERE、GROUP BY、HAVING、ORDER BY、LIMIT、子查询、JOIN、窗口函数等） |
| `SHOW TABLES LIKE` | 列出匹配模式的索引 |
| `DESCRIBE TABLES LIKE ... COLUMNS LIKE` | 查看索引字段映射 |
| `DELETE` | 按条件删除文档（默认禁用，需手动开启） |

### SHOW 语句

列出匹配模式的索引：

```sql
SHOW TABLES LIKE 'account%'
```

### DESCRIBE 语句

查看索引字段映射，可用 `COLUMNS LIKE` 过滤字段名：

```sql
DESCRIBE TABLES LIKE 'accounts' COLUMNS LIKE '%name%'
```

### DELETE 语句

> ⚠️ DELETE 默认禁用，需设置 `easysearch.sql.delete.enabled: true` 后方可使用。

```sql
DELETE FROM accounts WHERE age > 50
```

## 数据类型映射

| SQL 类型 | Easysearch 类型 | 说明 |
|---------|----------------|------|
| `BOOLEAN` | `boolean` | 布尔值 |
| `BYTE` | `byte` | 8 位整数 |
| `SHORT` | `short` | 16 位整数 |
| `INTEGER` | `integer` | 32 位整数 |
| `LONG` | `long` | 64 位整数 |
| `FLOAT` | `float` | 单精度浮点数 |
| `DOUBLE` | `double` | 双精度浮点数 |
| `STRING` | `keyword` | 精确值字符串 |
| `TEXT` | `text` | 全文分析字符串 |
| `DATE` | `date` | 日期 |
| `TIME` | — | 时间 |
| `DATETIME` / `TIMESTAMP` | `date` | 日期时间 |
| `OBJECT` | `object` | 嵌套 JSON 对象 |
| `NESTED` | `nested` | 嵌套文档数组 |
| `GEO_POINT` | `geo_point` | 经纬度坐标 |
| `IP` | `ip` | IP 地址 |
| `BINARY` | `binary` | 二进制数据 |

类型之间的隐式转换遵循层级关系：`BYTE → SHORT → INTEGER → LONG → FLOAT → DOUBLE`。可通过 `CAST()` 函数进行显式类型转换。

## 插件设置

| 设置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `easysearch.sql.enabled` | Boolean | `true` | 启用/禁用 SQL 插件 |
| `easysearch.sql.slowlog` | Integer | `2`（秒） | 慢查询日志阈值 |
| `easysearch.sql.cursor.keep_alive` | TimeValue | `1m` | 游标 Scroll 上下文保持时间 |
| `easysearch.sql.delete.enabled` | Boolean | `false` | 是否允许 DELETE 语句 |
| `easysearch.sql.query.size_limit` | Integer | `200` | 默认结果集大小上限 |
| `easysearch.sql.query.response.format` | String | `jdbc` | 默认响应格式 |
| `easysearch.sql.metrics.rollingwindow` | Long | `3600`（秒） | 指标统计滚动窗口 |
| `easysearch.sql.metrics.rollinginterval` | Long | `60`（秒） | 指标统计滚动间隔 |

以上设置均为动态设置，支持运行时更新：

```json
PUT /_sql/settings
{
  "persistent": {
    "easysearch.sql.enabled": true,
    "easysearch.sql.slowlog": 5,
    "easysearch.sql.query.size_limit": 500
  }
}
```

## 文档导航

1. [基础查询]({{< relref "basics" >}})：SELECT、FROM、WHERE、ORDER BY、LIMIT 等基本语法
2. [全文搜索]({{< relref "fulltext" >}})：在 SQL 中使用 MATCH、SCORE 等全文搜索函数
3. [复杂查询]({{< relref "complex" >}})：子查询、JOIN、CASE WHEN 等高级语法
4. [聚合查询]({{< relref "aggregations" >}})：GROUP BY、HAVING、聚合函数、FILTER 子句
5. [内置函数]({{< relref "functions" >}})：数学、字符串、日期、条件、类型转换函数参考
6. [SQL-JDBC]({{< relref "sql-jdbc" >}})：通过 JDBC 驱动连接 Easysearch


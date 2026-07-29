---
title: "通过 SQL 访问 Easysearch"
date: 0001-01-01
description: "面向 BI/报表与临时查询场景，使用 SQL 语法访问 Easysearch 的基础架构。"
summary: "通过 SQL 访问 Easysearch #  Easysearch 提供 SQL 查询接口，允许你用熟悉的 SQL 语法查询索引中的数据。这对于 BI 分析师、报表开发者和需要快速进行数据探索的场景非常方便。
相关指南（先读这些） #    SQL 查询功能  REST Client API  能力概览 #     功能 支持情况 说明     SELECT ✅ 支持列选择、别名   WHERE ✅ 支持 AND/OR/NOT、比较运算、LIKE、IN   ORDER BY ✅ 支持多列排序   GROUP BY ✅ 支持分组聚合   聚合函数 ✅ COUNT、SUM、AVG、MIN、MAX 等   LIMIT ✅ 支持分页   HAVING ✅ 聚合后过滤   JOIN ⚠️ 有限支持   子查询 ⚠️ 部分场景支持   DDL (CREATE/DROP) ❌ 不支持，使用 REST API 管理索引    使用方式 #  REST API 直接查询 #  POST _sql { &#34;query&#34;: &#34;SELECT speaker, COUNT(*) as cnt FROM shakespeare GROUP BY speaker ORDER BY cnt DESC LIMIT 5&#34; } 返回表格格式的结果："
---


# 通过 SQL 访问 Easysearch

Easysearch 提供 SQL 查询接口，允许你用熟悉的 SQL 语法查询索引中的数据。这对于 BI 分析师、报表开发者和需要快速进行数据探索的场景非常方便。

## 相关指南（先读这些）

- [SQL 查询功能]({{< relref "../../features/sql/" >}})
- [REST Client API]({{< relref "./client-api/client-api.md" >}})

## 能力概览

| 功能             | 支持情况 | 说明                                     |
| ---------------- | -------- | ---------------------------------------- |
| SELECT           | ✅       | 支持列选择、别名                          |
| WHERE            | ✅       | 支持 AND/OR/NOT、比较运算、LIKE、IN      |
| ORDER BY         | ✅       | 支持多列排序                              |
| GROUP BY         | ✅       | 支持分组聚合                              |
| 聚合函数         | ✅       | COUNT、SUM、AVG、MIN、MAX 等              |
| LIMIT            | ✅       | 支持分页                                  |
| HAVING           | ✅       | 聚合后过滤                                |
| JOIN             | ⚠️       | 有限支持                                  |
| 子查询           | ⚠️       | 部分场景支持                              |
| DDL (CREATE/DROP)| ❌       | 不支持，使用 REST API 管理索引             |

## 使用方式

### REST API 直接查询

```
POST _sql
{
  "query": "SELECT speaker, COUNT(*) as cnt FROM shakespeare GROUP BY speaker ORDER BY cnt DESC LIMIT 5"
}
```

返回表格格式的结果：

```json
{
  "schema": [
    { "name": "speaker", "type": "text" },
    { "name": "cnt", "type": "long" }
  ],
  "datarows": [
    ["GLOUCESTER", 556],
    ["HAMLET", 532],
    ["KING HENRY V", 428],
    ["KING RICHARD II", 405],
    ["DUKE OF YORK", 394]
  ],
  "total": 5,
  "size": 5
}
```

### 查看执行计划

使用 `explain` API 查看 SQL 语句转换后的 DSL：

```
POST _sql/_explain
{
  "query": "SELECT * FROM shakespeare WHERE speaker = 'HAMLET' LIMIT 10"
}
```

这会返回等价的 Query DSL JSON，帮助你理解和优化查询。

## 常用查询示例

### 条件过滤

```sql
SELECT play_name, speaker, text_entry
FROM shakespeare
WHERE play_name = 'Hamlet'
  AND speaker IN ('HAMLET', 'HORATIO')
LIMIT 20
```

### 聚合统计

```sql
SELECT play_name, COUNT(*) as lines, COUNT(DISTINCT speaker) as characters
FROM shakespeare
GROUP BY play_name
ORDER BY lines DESC
LIMIT 10
```

### 日期范围过滤

```sql
SELECT *
FROM logs
WHERE @timestamp >= '2024-01-01'
  AND @timestamp < '2024-02-01'
ORDER BY @timestamp DESC
LIMIT 100
```

## JDBC 驱动

Easysearch 提供 JDBC 驱动，可以让标准的数据库工具（如 DBeaver、DataGrip）直接连接：

```
jdbc:easysearch://https://localhost:9200
```

### 配合 BI 工具

| 工具          | 连接方式                            | 说明                              |
| ------------- | ----------------------------------- | --------------------------------- |
| INFINI Console | 内置 SQL 编辑器                    | 开箱即用                          |
| DBeaver        | JDBC 驱动                          | 通用数据库管理工具                |
| Grafana        | Elasticsearch 数据源               | 兼容 Elasticsearch 数据源插件     |
| Superset       | 见 [Superset 集成]({{< relref "./superset.md" >}}) | 需要配置数据源     |

## 注意事项

- SQL 查询最终会被转换为 Easysearch 的 Query DSL 执行，部分复杂的 SQL 语法可能无法完全支持
- 对于高频生产查询，建议直接使用 Query DSL 以获得更精细的控制
- 全文搜索功能（如 MATCH）可以在 WHERE 子句中使用：`WHERE MATCH(field, 'keyword')`



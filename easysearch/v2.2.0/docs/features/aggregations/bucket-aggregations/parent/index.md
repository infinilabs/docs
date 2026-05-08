---
title: "父文档聚合（Parent）"
date: 0001-01-01
summary: "父文档聚合 #  parent 聚合是一个分组聚合，根据您索引中定义的父子关系创建一个包含父级文档的分组。此聚合使您能够对具有相同匹配子级文档的父级文档执行分析，从而实现强大的层次结构数据分析。
parent 聚合与 join 字段类型一起工作，该字段类型在同一个索引中的文档内建立父子关系。
parent 聚合识别具有匹配子文档的父文档，而 children 聚合识别匹配特定子关系的子文档。这两种聚合都使用子关系名称作为输入。
相关指南（先读这些） #    聚合基础  Parent-Child 建模  聚合场景实践  参数说明 #  parent 聚合具有以下参数：
   参数 必需/可选 数据类型 描述     type	 必填 String join 字段中的子类型名称。    参考样例 #  以下示例构建了一个包含三名员工的小公司数据库。每个员工记录都与一个父部门记录存在 join 子关系。
首先，创建一个 company 索引，其中包含一个 join 字段，该字段将部门（父级）映射到员工（子级）：
PUT /company { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;join_field&#34;: { &#34;type&#34;: &#34;join&#34;, &#34;relations&#34;: { &#34;department&#34;: &#34;employee&#34; } }, &#34;department_name&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;employee_name&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;salary&#34;: { &#34;type&#34;: &#34;double&#34; }, &#34;hire_date&#34;: { &#34;type&#34;: &#34;date&#34; } } } } 接下来，用三个部门和三个员工填充数据。父子关系在以下表格中展示。"
---


# 父文档聚合

`parent` 聚合是一个分组聚合，根据您索引中定义的父子关系创建一个包含父级文档的分组。此聚合使您能够对具有相同匹配子级文档的父级文档执行分析，从而实现强大的层次结构数据分析。

`parent` 聚合与 `join` 字段类型一起工作，该字段类型在同一个索引中的文档内建立父子关系。

`parent` 聚合识别具有匹配子文档的父文档，而 `children` 聚合识别匹配特定子关系的子文档。这两种聚合都使用子关系名称作为输入。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`parent` 聚合具有以下参数：

| 参数   | 必需/可选 | 数据类型 | 描述                        |
| ------ | --------- | -------- | --------------------------- |
| `type	` | 必填      | String   | `join` 字段中的子类型名称。 |

## 参考样例

以下示例构建了一个包含三名员工的小公司数据库。每个员工记录都与一个父部门记录存在 `join` 子关系。

首先，创建一个 `company` 索引，其中包含一个 `join` 字段，该字段将部门（父级）映射到员工（子级）：

```
PUT /company
{
  "mappings": {
    "properties": {
      "join_field": {
        "type": "join",
        "relations": {
          "department": "employee"
        }
      },
      "department_name": {
        "type": "keyword"
      },
      "employee_name": {
        "type": "keyword"
      },
      "salary": {
        "type": "double"
      },
      "hire_date": {
        "type": "date"
      }
    }
  }
}

```

接下来，用三个部门和三个员工填充数据。父子关系在以下表格中展示。

| 部门（父关系） | 员工（子关系）                |
| -------------- | ----------------------------- |
| Accounting     | Abel Anderson, Betty Billings |
| Engineering    | Carl Carter                   |
| HR             | none                          |

routing 参数确保父级和子级文档存储在同一个分片上：

```
POST _bulk?routing=1
{ "create": { "_index": "company", "_id": "1" } }
{ "type": "department", "department_name": "Accounting", "join_field": "department" }
{ "create": { "_index": "company", "_id": "2" } }
{ "type": "department", "department_name": "Engineering", "join_field": "department" }
{ "create": { "_index": "company", "_id": "3" } }
{ "type": "department", "department_name": "HR", "join_field": "department" }
{ "create": { "_index": "company", "_id": "4" } }
{ "type": "employee", "employee_name": "Abel Anderson", "salary": 120000, "hire_date": "2024-04-04", "join_field": { "name": "employee",  "parent": "1" } }
{ "create": { "_index": "company", "_id": "5" } }
{ "type": "employee", "employee_name": "Betty Billings", "salary": 140000, "hire_date": "2023-05-05", "join_field": { "name": "employee",  "parent": "1" } }
{ "create": { "_index": "company", "_id": "6" } }
{ "type": "employee", "employee_name": "Carl Carter", "salary": 140000, "hire_date": "2020-06-06",  "join_field": { "name": "employee",  "parent": "2" } }

```

最后，运行一个聚合操作，统计与一个或多个员工存在父子关系的所有部门：

```
GET /company/_search
{
  "size": 0,
  "aggs": {
    "all_departments": {
      "parent": {
        "type": "employee"
      },
      "aggs": {
        "departments": {
          "terms": {
            "field": "department_name"
          }
        }
      }
    }
  }
}

```

返回内容

`all_departments` 父聚合返回所有包含员工子文档的部门。请注意，人力资源部门没有体现：

```
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 6,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_departments": {
      "doc_count": 2,
      "departments": {
        "doc_count_error_upper_bound": 0,
        "sum_other_doc_count": 0,
        "buckets": [
          {
            "key": "Accounting",
            "doc_count": 1
          },
          {
            "key": "Engineering",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```


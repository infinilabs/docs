---
title: "子文档聚合（Children）"
date: 0001-01-01
summary: "子文档聚合 #  children 聚合是一种存储分组聚合，它根据索引中定义的父子关系创建包含子文档的单个存储分组。
children 聚合与 join 字段类型配合使用，以聚合与父文档关联的子文档。
children 聚合标识与特定子关系名称匹配的子文档，而 parent 聚合标识具有匹配子文档的父文档。这两个聚合都采用子关系名称作为输入。
相关指南（先读这些） #    聚合基础  Parent-Child 建模  聚合场景实践  参数说明 #  children 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     type 必填 String join 字段中的子类型的名称。这标识了要使用的父子关系。    参考样例 #  以下示例构建一个包含三名员工的小型公司数据库。每个员工记录都与父部门记录具有子联接关系。
首先，创建一个 company 索引，其中包含一个将部门（父级）映射到员工（子级）的联接字段：
PUT /company { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;join_field&#34;: { &#34;type&#34;: &#34;join&#34;, &#34;relations&#34;: { &#34;department&#34;: &#34;employee&#34; } }, &#34;department_name&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;employee_name&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;salary&#34;: { &#34;type&#34;: &#34;double&#34; }, &#34;hire_date&#34;: { &#34;type&#34;: &#34;date&#34; } } } } 接下来，使用三个部门和三名员工填充数据。下表显示了父子分配。"
---


# 子文档聚合

`children` 聚合是一种存储分组聚合，它根据索引中定义的父子关系创建包含子文档的单个存储分组。

`children` 聚合与 `join` 字段类型配合使用，以聚合与父文档关联的子文档。

`children` 聚合标识与特定子关系名称匹配的子文档，而 `parent` 聚合标识具有匹配子文档的父文档。这两个聚合都采用子关系名称作为输入。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`children` 聚合采用以下参数。

| 参数   | 必需/可选 | 数据类型 | 描述                                                  |
| ------ | --------- | -------- | ----------------------------------------------------- |
| `type` | 必填      | String   | join 字段中的子类型的名称。这标识了要使用的父子关系。 |

## 参考样例

以下示例构建一个包含三名员工的小型公司数据库。每个员工记录都与父部门记录具有子联接关系。

首先，创建一个 `company` 索引，其中包含一个将部门（父级）映射到员工（子级）的联接字段：

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

接下来，使用三个部门和三名员工填充数据。下表显示了父子分配。

| 部门（父关系） | 员工（子关系）                |
| -------------- | ----------------------------- |
| Accounting     | Abel Anderson, Betty Billings |
| Engineering    | Carl Carter                   |
| HR             | none                          |

`routing` 参数可确保父文档和子文档存储在同一分片上，这是父子关系在 Easysearch 中正常运行所必需的：

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

以下请求查询所有部门，然后筛选名为 `Accounting` 的部门。然后，它使用子关联聚合来选择与会计部门具有子关系的两个单据。最后，`avg` 子关联聚合返回会计员工工资的平均值：

```
GET /company/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "join_field": "department"
          }
        },
        {
          "term": {
            "department_name": "Accounting"
          }
        }
      ]
    }
  },
  "aggs": {
    "acc_employees": {
      "children": {
        "type": "employee"
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    }
  }
}
```

返回内容包含所选部门存储分组，查找部门的员工类型子级，并计算其工资的平均值 ：

```
{
  "took": 379,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "acc_employees": {
      "doc_count": 2,
      "avg_salary": {
        "value": 110000
      }
    }
  }
}

```


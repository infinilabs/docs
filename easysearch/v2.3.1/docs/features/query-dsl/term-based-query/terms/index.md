---
title: "Terms 查询"
date: 0001-01-01
summary: "Terms 查询 #  使用 terms 查询在同一字段中搜索多个词项。例如，以下查询搜索具有 ID 61809 和 61810 的文档：
相关指南（先读这些） #    结构化搜索  Query DSL 基础  GET shakespeare/_search { &#34;query&#34;: { &#34;terms&#34;: { &#34;line_id&#34;: [ &#34;61809&#34;, &#34;61810&#34; ] } } } 如果文档与数组中的任何词项匹配，则会返回该文档。
默认情况下， terms 查询中允许的最大词项数量为 65,536。要更改最大词项数量，请更新 index.max_terms_count 设置。
 为了更好的查询性能，请传递包含已排序词项的长期数组（按 UTF-8 字节值升序排序）。
  根据高亮器类型和查询中词项的数量，高亮显示词项查询结果的能力可能无法保证。
 参数说明 #  该查询接受以下参数。所有参数都是可选的。
   参数 数据类型 描述     &lt;field&gt; String 要搜索的字段。只有当文档的字段值与查询中至少一个词项完全匹配（包括正确的空格和大小写）时，该文档才会出现在结果中。   boost Float 一个浮点数值，用于指定该字段对相关性分数的权重。大于 1."
---


# Terms 查询

使用 `terms` 查询在同一字段中搜索多个词项。例如，以下查询搜索具有 `ID` `61809` 和 `61810` 的文档：

## 相关指南（先读这些）

- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

```
GET shakespeare/_search
{
  "query": {
    "terms": {
      "line_id": [
        "61809",
        "61810"
      ]
    }
  }
}

```

如果文档与数组中的任何词项匹配，则会返回该文档。

默认情况下， `terms` 查询中允许的最大词项数量为 65,536。要更改最大词项数量，请更新 `index.max_terms_count` 设置。

> 为了更好的查询性能，请传递包含已排序词项的长期数组（按 UTF-8 字节值升序排序）。

> 根据高亮器类型和查询中词项的数量，高亮显示词项查询结果的能力可能无法保证。

## 参数说明

该查询接受以下参数。所有参数都是可选的。

| 参数         | 数据类型 | 描述                                                                                                                                    |
| ------------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `<field>`    | String   | 要搜索的字段。只有当文档的字段值与查询中至少一个词项完全匹配（包括正确的空格和大小写）时，该文档才会出现在结果中。                      |
| `boost`      | Float    | 一个浮点数值，用于指定该字段对相关性分数的权重。大于 1.0 的值会增加字段的权重。介于 0.0 和 1.0 之间的值会降低字段的权重。默认值为 1.0。 |
| `_name`      | String   | 查询标签的查询名称。可选。                                                                                                              |
| `value_type` | String   | 指定用于过滤的值类型。有效值为 `default` 和 `bitmap` 。如果省略，则值默认为 `default` 。                                                |

## 条件查找

条件查找功能会检索单个文档的字段值并将其用作搜索词。您可以使用条件查找功能来搜索大量词项。

要使用条件查找功能，您必须启用 \_source 映射字段，因为条件查找功能会从文档中获取值。默认情况下， \_source 字段处于启用状态。

条件查找会尝试从本地数据节点上的分片获取文档字段值。因此，使用具有单个主分片且在所有适用数据节点上都有完整副本的索引可以减少网络流量。

### 参考用例

例如，创建一个包含学生数据的索引，将 `student_id` 映射为 `keyword` ：

```
PUT students
{
  "mappings": {
    "properties": {
      "student_id": { "type": "keyword" }
    }
  }
}

```

接下来，索引与学生对应的三个文档：

```
PUT students/_doc/1
{
  "name": "Jane Doe",
  "student_id" : "111"
}

PUT students/_doc/2
{
  "name": "Mary Major",
  "student_id" : "222"
}

PUT students/_doc/3
{
  "name": "John Doe",
  "student_id" : "333"
}

```

创建一个单独的索引，其中包含班级信息，包括班级名称和与该班级注册的学生对应的学生 ID 数组：

```
PUT classes/_doc/101
{
  "name": "CS101",
  "enrolled" : ["111" , "222"]
}

```

要搜索参加 `CS101` 课程的学生，请指定与该课程对应的文档的文档 ID、该文档的索引以及词项所在字段的路径：

```
GET students/_search
{
  "query": {
    "terms": {
      "student_id": {
        "index": "classes",
        "id": "101",
        "path": "enrolled"
      }
    }
  }
}
```

返回内容包含 `students` 索引中 ID 与 `enrolled` 数组中的一个值匹配的每个学生的文档：

```
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "students",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Jane Doe",
          "student_id": "111"
        }
      },
      {
        "_index": "students",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Mary Major",
          "student_id": "222"
        }
      }
    ]
  }
}
```

### 示例：嵌套字段

第二个示例演示了如何查询嵌套字段。考虑以下文档的索引：

```
PUT classes/_doc/102
{
  "name": "CS102",
  "enrolled_students" : {
    "id_list" : ["111" , "333"]
  }
}

```

要搜索就读 `CS102` 的学生，请使用点路径符号指定 `path` 参数中字段的完整路径：

```
GET students/_search
{
  "query": {
    "terms": {
      "student_id": {
        "index": "classes",
        "id": "102",
        "path": "enrolled_students.id_list"
      }
    }
  }
}
```

返回内容包含匹配的文档：

```
{
  "took": 18,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "students",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Jane Doe",
          "student_id": "111"
        }
      },
      {
        "_index": "students",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "John Doe",
          "student_id": "333"
        }
      }
    ]
  }
}
```

### 参数说明

下表列出了条件查找参数。

| 参数      | 数据类型 | 描述                                                                                           |
| --------- | -------- | ---------------------------------------------------------------------------------------------- |
| `index`   | String   | 用于获取字段值的索引的名称。必填。                                                             |
| `id`      | String   | 要从中获取字段值的文档的文档 ID。必填。                                                        |
| `query`   | Object   | 用于选择多个文档并从中获取字段值的查询对象。如果未提供 id ，则为必填项。                       |
| `path`    | String   | 要从中获取字段值的字段名称。使用点路径表示法指定嵌套字段。必填。                               |
| `routing` | String   | 用于从中获取字段值的文档的自定义路由值。可选。如果在索引文档时提供了自定义路由值，则为必填项。 |
| `store`   | Boolean  | 是否在存储字段（而非 `_source` 上执行查找。可选。                                              |


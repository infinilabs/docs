---
title: "Terms Set 查询"
date: 0001-01-01
summary: "Terms Set 查询 #  使用 terms_set 查询，您可以在指定字段中搜索匹配一定数量的精确词的文档。与 terms 查询类似，您可以指定返回文档所需的匹配词的最小数量。您可以直接在索引字段中指定这个数量，也可以通过脚本指定。
相关指南（先读这些） #    结构化搜索  查询 DSL 基础  例如，假设有一个索引，其中包含学生的姓名和他们所选的课程。在设置该索引的映射时，您需要提供一个数值字段，以指定返回文档所需的最小匹配项数量：
PUT students { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;classes&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;min_required&#34;: { &#34;type&#34;: &#34;integer&#34; } } } } 接下来，索引两个与学生相关的文档：
PUT students/_doc/1 { &#34;name&#34;: &#34;Mary Major&#34;, &#34;classes&#34;: [ &#34;CS101&#34;, &#34;CS102&#34;, &#34;MATH101&#34; ], &#34;min_required&#34;: 2 } PUT students/_doc/2 { &#34;name&#34;: &#34;John Doe&#34;, &#34;classes&#34;: [ &#34;CS101&#34;, &#34;MATH101&#34;, &#34;ENG101&#34; ], &#34;min_required&#34;: 2 } 现在搜索已经修读了以下至少两门课程的学生： CS101 ， CS102 ， MATH101 ："
---


# Terms Set 查询

使用 `terms_set` 查询，您可以在指定字段中搜索匹配一定数量的精确词的文档。与 `terms` 查询类似，您可以指定返回文档所需的匹配词的最小数量。您可以直接在索引字段中指定这个数量，也可以通过脚本指定。

## 相关指南（先读这些）

- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

例如，假设有一个索引，其中包含学生的姓名和他们所选的课程。在设置该索引的映射时，您需要提供一个数值字段，以指定返回文档所需的最小匹配项数量：

```
PUT students
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword"
      },
      "classes": {
        "type": "keyword"
      },
      "min_required": {
        "type": "integer"
      }
    }
  }
}

```

接下来，索引两个与学生相关的文档：

```
PUT students/_doc/1
{
  "name": "Mary Major",
  "classes": [ "CS101", "CS102", "MATH101" ],
  "min_required": 2
}

PUT students/_doc/2
{
  "name": "John Doe",
  "classes": [ "CS101", "MATH101", "ENG101" ],
  "min_required": 2
}

```

现在搜索已经修读了以下至少两门课程的学生： `CS101` ， `CS102` ， `MATH101` ：

```
GET students/_search
{
  "query": {
    "terms_set": {
      "classes": {
        "terms": [ "CS101", "CS102", "MATH101" ],
        "minimum_should_match_field": "min_required"
      }
    }
  }
}

```

返回内容中包含了两份文档：

```
{
  "took" : 44,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.4544616,
    "hits" : [
      {
        "_index" : "students",
        "_id" : "1",
        "_score" : 1.4544616,
        "_source" : {
          "name" : "Mary Major",
          "classes" : [
            "CS101",
            "CS102",
            "MATH101"
          ],
          "min_required" : 2
        }
      },
      {
        "_index" : "students",
        "_id" : "2",
        "_score" : 0.5013843,
        "_source" : {
          "name" : "John Doe",
          "classes" : [
            "CS101",
            "MATH101",
            "ENG101"
          ],
          "min_required" : 2
        }
      }
    ]
  }
}
```

要指定文档应与脚本匹配的最小词项数，请在 `minimum_should_match_script` 字段中提供脚本：

```
GET students/_search
{
  "query": {
    "terms_set": {
      "classes": {
        "terms": [ "CS101", "CS102", "MATH101" ],
        "minimum_should_match_script": {
          "source": "Math.min(params.num_terms, doc['min_required'].value)"
        }
      }
    }
  }
}

```

## 参数说明

查询接受字段的名称 （`<field>`） 作为顶级参数：

```

GET _search
{
  "query": {
    "terms_set": {
      "<field>": {
        "terms": [ "term1", "term2" ],
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除 `terms` 外，所有参数均为可选参数。

| 参数                          | 数据类型         | 描述                                                                                                                                                 |
| ----------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `terms`                       | Array of strings | 要在 <field> 中指定的字段中搜索的词项数组。仅当所需的词项数与文档的字段值完全匹配且间距和大小写正确时，才会在结果中返回文档。                        |
| `minimum_should_match_field`  | String           | 数值字段的名称，用于指定在结果中返回文档所需的匹配项数。必须指定 `minimum_should_match_field` 或 `minimum_should_match_script`，但不能同时指定两者。 |
| `minimum_should_match_script` | String           | 返回结果中返回文档所需的匹配词数的脚本。必须指定 `minimum_should_match_field` 或 `minimum_should_match_script`，但不能同时指定两者。                 |
| `boost`                       | Float            | 一个浮点值，用于指定此字段相对于相关性分数的权重。值高于 1.0 会增加字段的相关性。值介于 0.0 和 1.0 之间会降低字段的相关性。默认值为 1.0。            |


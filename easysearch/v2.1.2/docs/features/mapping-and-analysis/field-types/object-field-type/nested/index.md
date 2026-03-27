---
title: "嵌套字段类型（Nested）"
date: 0001-01-01
summary: "Nested 字段类型 #  nested 字段类型是一种特殊的 对象字段类型，用于将数组中的对象作为独立文档索引，避免扁平化导致的&quot;笛卡尔积假匹配&quot;问题。
相关指南（先读这些） #    Nested 建模  映射模式  扁平化 vs Nested #  任何对象字段都可以包含一个对象数组。默认情况下，数组中的每个对象都会被动态映射为对象字段类型并以扁平化形式存储。这意味着数组中的对象会被分解成单独的字段，每个字段在所有对象中的值会被存储在一起。有时需要使用 Nested 嵌套类型来将嵌套对象作为一个整体保存，以便您可以对其关联性执行搜索。
扁平化形式 #  默认情况下，每个嵌套对象都被动态映射为对象字段类型。任何对象字段都可以包含一个对象数组。
PUT testindex1/_doc/100 { &#34;patients&#34;: [ {&#34;name&#34;: &#34;John Doe&#34;, &#34;age&#34;: 56, &#34;smoker&#34;: true}, {&#34;name&#34;: &#34;Mary Major&#34;, &#34;age&#34;: 85, &#34;smoker&#34;: false} ] } 当这些对象被存储时，它们会被扁平化，因此它们的内部表示形式具有每个字段的所有值的数组：
{ &#34;patients.name&#34;: [&#34;John Doe&#34;, &#34;Mary Major&#34;], &#34;patients.age&#34;: [56, 85], &#34;patients.smoker&#34;: [true, false] } 一些查询会在这种表示形式中正确工作。如果您搜索年龄大于 75 或者 吸烟的病人 &quot;patients.smoker&quot;: true，文档 id 100 应该匹配。"
---


# Nested 字段类型

`nested` 字段类型是一种特殊的[对象字段类型](./object.md)，用于将数组中的对象作为独立文档索引，避免扁平化导致的"笛卡尔积假匹配"问题。

## 相关指南（先读这些）

- [Nested 建模]({{< relref "/docs/best-practices/data-modeling/nested.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 扁平化 vs Nested

任何对象字段都可以包含一个对象数组。默认情况下，数组中的每个对象都会被动态映射为对象字段类型并以扁平化形式存储。这意味着数组中的对象会被分解成单独的字段，每个字段在所有对象中的值会被存储在一起。有时需要使用 Nested 嵌套类型来将嵌套对象作为一个整体保存，以便您可以对其关联性执行搜索。

## 扁平化形式

默认情况下，每个嵌套对象都被动态映射为对象字段类型。任何对象字段都可以包含一个对象数组。

```json
PUT testindex1/_doc/100
{
  "patients": [
    {"name": "John Doe", "age": 56, "smoker": true},
    {"name": "Mary Major", "age": 85, "smoker": false}
  ]
}
```

当这些对象被存储时，它们会被扁平化，因此它们的内部表示形式具有每个字段的所有值的数组：

```json
{
  "patients.name": ["John Doe", "Mary Major"],
  "patients.age": [56, 85],
  "patients.smoker": [true, false]
}
```

一些查询会在这种表示形式中正确工作。如果您搜索年龄大于 75 或者 吸烟的病人 `"patients.smoker": true`，文档 id 100 应该匹配。

```json
GET testindex1/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "patients.smoker": true
          }
        },
        {
          "range": {
            "patients.age": {
              "gte": 75
            }
          }
        }
      ]
    }
  }
}
```

查询正确地返回文档 id 100：

```json
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
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.3616575,
    "hits": [
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "100",
        "_score": 1.3616575,
        "_source": {
          "patients": [
            {
              "name": "John Doe",
              "age": "56",
              "smoker": true
            },
            {
              "name": "Mary Major",
              "age": "85",
              "smoker": false
            }
          ]
        }
      }
    ]
  }
}
```

或者，如果您搜索年龄大于 75 并且是吸烟的病人，文档 100 不应该匹配。

```json
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "patients.smoker": true
          }
        },
        {
          "range": {
            "patients.age": {
              "gte": 75
            }
          }
        }
      ]
    }
  }
}
```

然而，这个查询仍然错误地返回文档 100。这是因为当创建每个字段的值数组时，年龄和吸烟患者之间的关联关系丢失了。

## Nested 嵌套字段类型

Nested 嵌套对象虽然作为独立的文档存储，但是父对象持有对其子对象的引用，这样就保留了子对象之间的关联关系。要将对象标记为嵌套类型，需要创建一个带有嵌套字段类型的映射。

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "patients": {
        "type": "nested"
      }
    }
  }
}
```

然后，索引一个带有嵌套字段类型的文档：

```json
PUT testindex1/_doc/100
{
  "patients": [
    {"name": "John Doe", "age": 56, "smoker": true},
    {"name": "Mary Major", "age": 85, "smoker": false}
  ]
}
```

您可以使用以下嵌套查询来搜索年龄大于 75 或者 吸烟的患者：

```json
GET testindex1/_search
{
  "query": {
    "nested": {
      "path": "patients",
      "query": {
        "bool": {
          "should": [
            {
              "term": {
                "patients.smoker": true
              }
            },
            {
              "range": {
                "patients.age": {
                  "gte": 75
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

查询正确地返回了两个患者：

```json
{
  "took": 7,
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
    "max_score": 0.8465736,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "100",
        "_score": 0.8465736,
        "_source": {
          "patients": [
            {
              "name": "John Doe",
              "age": 56,
              "smoker": true
            },
            {
              "name": "Mary Major",
              "age": 85,
              "smoker": false
            }
          ]
        }
      }
    ]
  }
}
```

您可以使用以下嵌套查询来搜索年龄大于 75 并且 吸烟的患者：

```json
GET testindex1/_search
{
  "query": {
    "nested": {
      "path": "patients",
      "query": {
        "bool": {
          "must": [
            {
              "term": {
                "patients.smoker": true
              }
            },
            {
              "range": {
                "patients.age": {
                  "gte": 75
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

如预期一样，前面的查询没有返回任何结果：

```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

## 参数说明

下表列出了对象字段类型接受的参数。所有参数都是可选的。

| 参数                | 说明                                                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `dynamic`           | 指定是否可以动态地向对象添加新字段。有效值为 `true`、`false`、`strict` 和 `strict_allow_templates`。默认为 `true`。 |
| `include_in_parent` | 一个布尔值，指定子嵌套对象中的所有字段是否也应以扁平化形式添加到父文档中。默认为 `false`。                          |
| `include_in_root`   | 一个布尔值，指定子嵌套对象中的所有字段是否也应以扁平化形式添加到根文档中。默认为 `false`。                          |
| `properties`        | 此对象的字段，可以是任何支持的类型。如果 `dynamic` 设置为 `true`，则可以动态地向此对象添加新属性。                  |


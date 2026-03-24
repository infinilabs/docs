---
title: "动态映射参数（Dynamic）"
date: 0001-01-01
summary: "Dynamic 参数 #  dynamic 参数指定是否可以动态地将新检测到的字段添加到映射中。它接受下表中列出的参数。
   参数 描述     true 指定可以动态地将新字段添加到映射中。默认值为 true。   false 指定不能动态地将新字段添加到映射中。如果检测到新字段，则不会对其进行索引或搜索，但可以从 _source 字段中检索。   strict 当检测到文档中有新字段时，索引操作失败，抛出异常。    相关指南（先读这些） #    映射基础  映射模式  示例：创建 dynamic 设置为 true 的索引 #  通过以下命令创建一个 dynamic 设置为 true 的索引：
PUT testindex1 { &#34;mappings&#34;: { &#34;dynamic&#34;: true } } 通过以下命令，索引一个包含两个字符串字段的对象字段 patient 的文档：
PUT testindex1/_doc/1 { &#34;patient&#34;: { &#34;name&#34; : &#34;John Doe&#34;, &#34;id&#34; : &#34;123456&#34; } } 通过以下命令确认映射按预期工作："
---


# Dynamic 参数

`dynamic` 参数指定是否可以动态地将新检测到的字段添加到映射中。它接受下表中列出的参数。

| 参数                     | 描述                                                                                                            |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- |
| `true`                   | 指定可以动态地将新字段添加到映射中。默认值为 `true`。                                                           |
| `false`                  | 指定不能动态地将新字段添加到映射中。如果检测到新字段，则不会对其进行索引或搜索，但可以从 `_source` 字段中检索。 |
| `strict`                 | 当检测到文档中有新字段时，索引操作失败，抛出异常。                                                              |

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 示例：创建 `dynamic` 设置为 `true` 的索引

通过以下命令创建一个 `dynamic` 设置为 `true` 的索引：

```json
PUT testindex1
{
  "mappings": {
    "dynamic": true
  }
}
```

通过以下命令，索引一个包含两个字符串字段的对象字段 `patient` 的文档：

```json
PUT testindex1/_doc/1
{
  "patient": {
    "name" : "John Doe",
    "id" : "123456"
  }
}
```

通过以下命令确认映射按预期工作：

```json
GET testindex1/_mapping
```

对象字段 `patient` 和两个子字段 `name` 和 `id` 被添加到映射中，如以下响应所示：

```json
{
  "testindex1": {
    "mappings": {
      "dynamic": "true",
      "properties": {
        "patient": {
          "properties": {
            "id": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        }
      }
    }
  }
}
```

## 示例：创建 `dynamic` 设置为 `false` 的索引

通过以下命令，创建一个具有显式映射且 `dynamic` 设置为 `false` 的索引：

```json
PUT testindex1
{
  "mappings": {
    "dynamic": false,
    "properties": {
      "patient": {
        "properties": {
          "id": {
            "type": "keyword"
          },
          "name": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

通过以下命令，索引一个文档，其中包含两个字符串字段的对象字段 `patient`和额外未映射的 `room` 和 `floor`

```json
PUT testindex1/_doc/1
{
  "patient": {
    "name" : "John Doe",
    "id" : "123456"
  },
  "room": "room1",
  "floor": "1"
}
```

通过以下命令确认映射是否按预期工作：

```json
GET testindex1/_mapping
```

以下返回内容显示新字段 `room` 和 `floor` 未添加到映射中，映射保持不变：

```json
{
  "testindex1": {
    "mappings": {
      "dynamic": "false",
      "properties": {
        "patient": {
          "properties": {
            "id": {
              "type": "keyword"
            },
            "name": {
              "type": "keyword"
            }
          }
        }
      }
    }
  }
}
```

## 示例：创建 `dynamic` 设置为 `strict` 的索引

通过以下命令，创建一个具有显式映射且 `dynamic` 设置为 `strict` 的索引：

```json
PUT testindex1
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "patient": {
        "properties": {
          "id": {
            "type": "keyword"
          },
          "name": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

通过以下命令，索引一个文档，其中包含两个字符串字段的对象字段 `patient` 和额外未映射的 `room` 和 `floor`

```json
PUT testindex1/_doc/1
{
  "patient": {
    "name" : "John Doe",
    "id" : "123456"
  },
  "room": "room1",
  "floor": "1"
}
```

注意会抛出异常，如下所示：

```json
{
  "error": {
    "root_cause": [
      {
        "type": "strict_dynamic_mapping_exception",
        "reason": "mapping set to strict, dynamic introduction of [room] within [_doc] is not allowed"
      }
    ],
    "type": "strict_dynamic_mapping_exception",
    "reason": "mapping set to strict, dynamic introduction of [room] within [_doc] is not allowed"
  },
  "status": 400
}
```




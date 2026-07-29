---
title: "对象字段类型（Object）"
date: 0001-01-01
summary: "Object 字段类型 #  object 字段类型包含一个 JSON 对象（一组名称/值对）。JSON 对象中的值可以是另一个 JSON 对象。在映射对象字段时不需要指定 object 作为类型，因为 object 是默认类型。
相关指南（先读这些） #    映射基础  映射模式  代码示例 #  创建一个带有对象字段的映射：
PUT testindex1/_mappings { &#34;properties&#34;: { &#34;patient&#34;: { &#34;properties&#34; : { &#34;name&#34; : { &#34;type&#34; : &#34;text&#34; }, &#34;id&#34; : { &#34;type&#34; : &#34;keyword&#34; } } } } } 索引一个包含对象字段的文档：
PUT testindex1/_doc/1 { &#34;patient&#34;: { &#34;name&#34; : &#34;John Doe&#34;, &#34;id&#34; : &#34;123456&#34; } } 嵌套对象在内部存储为扁平的键/值对。要引用嵌套对象中的字段，使用 parent field."
---


# Object 字段类型

`object` 字段类型包含一个 JSON 对象（一组名称/值对）。JSON 对象中的值可以是另一个 JSON 对象。在映射对象字段时不需要指定 `object` 作为类型，因为 `object` 是默认类型。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 代码示例

创建一个带有对象字段的映射：

```json
PUT testindex1/_mappings
{
    "properties": {
      "patient": {
        "properties" :
          {
            "name" : {
              "type" : "text"
            },
            "id" : {
              "type" : "keyword"
            }
          }
      }
    }
}
```

索引一个包含对象字段的文档：

```json
PUT testindex1/_doc/1
{
  "patient": {
    "name" : "John Doe",
    "id" : "123456"
  }
}
```

嵌套对象在内部存储为扁平的键/值对。要引用嵌套对象中的字段，使用 `parent field`.`child field`（例如，`patient.id`）。

搜索 ID 为 123456 的患者信息：

```json
GET testindex1/_search
{
  "query": {
    "term": {
      "patient.id": "123456"
    }
  }
}
```

### 参数

下表列出了对象字段类型接受的参数。所有参数都是可选的。

| 参数         | 描述                                                                                                                                                     |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dynamic`    | 指定是否可以动态地向对象添加新字段。有效值为 `true`、`false`、`strict` 和 `strict_allow_templates`。默认为 `true`。                                      |
| `enabled`    | 一个布尔值，指定是否应解析对象的 JSON 内容。如果 `enabled` 设置为 `false`，则对象的内容不会被索引或搜索，但仍可以从 \_source 字段中检索。默认为 `true`。 |
| `properties` | 此对象字段的属性（即子字段设置），可以是任何支持的类型。如果 `dynamic` 设置为 `true`，则可以动态地向此对象添加新属性。                                   |

### `dynamic` 参数

`dynamic` 参数指定是否可以向已经索引的对象动态添加新字段。

例如，您最初可以创建一个只有一个字段的 `patient` 对象的映射：

```json
PUT testindex1/_mappings
{
    "properties": {
      "patient": {
        "properties" :
          {
            "name" : {
              "type" : "text"
            }
          }
      }
    }
}
```

然后，您在 `patient` 中索引一个带有新 `id` 字段的文档：

```json
PUT testindex1/_doc/1
{
  "patient": {
    "name" : "John Doe",
    "id" : "123456"
  }
}
```

结果，`id` 字段被添加到映射中：

```json
{
  "testindex1": {
    "mappings": {
      "properties": {
        "patient": {
          "properties": {
            "id": {
              "type": "text"
            },
            "name": {
              "type": "text"
            }
          }
        }
      }
    }
  }
}
```

`dynamic` 参数有以下有效值：

| 值                       | 描述                                                                                                             |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| `true`                   | 可以动态地向映射添加新字段。这是默认值。                                                                         |
| `false`                  | 不能动态地向映射添加新字段。如果检测到新字段，它不会被索引或搜索。但是，仍然可以从 \_source 字段中检索它。       |
| `strict`                 | 当动态地向映射添加新字段时，会抛出异常。要向对象添加新字段，必须先将其添加到映射中。                             |
| `strict_allow_templates` | 如果新检测到的字段匹配映射中的任何预定义动态模板，则它们会被添加到映射中；如果它们不匹配任何模板，则会抛出异常。 |

> 内部对象会继承其父对象的 dynamic 参数值，除非它们声明了自己的 dynamic 参数值。


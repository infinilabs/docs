---
title: "元信息字段（_meta）"
date: 0001-01-01
summary: "_meta 元数据字段 #  _meta 字段是一个映射属性，允许您为索引映射附加自定义元数据。您的应用程序可以使用这些元数据来存储与您的用例相关的信息，如版本控制、所有权、分类或审计。
相关指南（先读这些） #    映射基础  元数据字段  用法 #  您可以在创建新索引或更新现有索引的映射时定义 _meta 字段，如以下示例所示：
PUT my-index { &#34;mappings&#34;: { &#34;_meta&#34;: { &#34;application&#34;: &#34;MyApp&#34;, &#34;version&#34;: &#34;1.2.3&#34;, &#34;author&#34;: &#34;John Doe&#34; }, &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;description&#34;: { &#34;type&#34;: &#34;text&#34; } } } } 在此示例中，添加了三个自定义元数据字段：application、version 和 author。您的应用程序可以使用这些字段来存储有关索引的任何相关信息，例如它所属的应用程序、应用程序版本或索引的作者。
您可以使用 Put Mapping API 操作更新 _meta 字段，如以下示例所示：
PUT my-index/_mapping { &#34;_meta&#34;: { &#34;application&#34;: &#34;MyApp&#34;, &#34;version&#34;: &#34;1.3.0&#34;, &#34;author&#34;: &#34;Jane Smith&#34; } } 检索元数据信息 #  您可以使用 Get Mapping API 操作检索索引的 _meta 信息，如以下示例所示："
---


# _meta 元数据字段

`_meta` 字段是一个映射属性，允许您为索引映射附加自定义元数据。您的应用程序可以使用这些元数据来存储与您的用例相关的信息，如版本控制、所有权、分类或审计。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [元数据字段]({{< relref "/docs/features/mapping-and-analysis/metadata-field/_index.md" >}})

## 用法

您可以在创建新索引或更新现有索引的映射时定义 `_meta` 字段，如以下示例所示：

```
PUT my-index
{
  "mappings": {
    "_meta": {
      "application": "MyApp",
      "version": "1.2.3",
      "author": "John Doe"
    },
    "properties": {
      "title": {
        "type": "text"
      },
      "description": {
        "type": "text"
      }
    }
  }
}
```

在此示例中，添加了三个自定义元数据字段：`application`、`version` 和 `author`。您的应用程序可以使用这些字段来存储有关索引的任何相关信息，例如它所属的应用程序、应用程序版本或索引的作者。

您可以使用 Put Mapping API 操作更新 `_meta` 字段，如以下示例所示：

```
PUT my-index/_mapping
{
  "_meta": {
    "application": "MyApp",
    "version": "1.3.0",
    "author": "Jane Smith"
  }
}
```

## 检索元数据信息

您可以使用 Get Mapping API 操作检索索引的 `_meta` 信息，如以下示例所示：

```
GET my-index/_mapping
```

返回包含 `_meta` 字段的完整索引映射：

```
{
  "my-index": {
    "mappings": {
      "_meta": {
        "application": "MyApp",
        "version": "1.3.0",
        "author": "Jane Smith"
      },
      "properties": {
        "description": {
          "type": "text"
        },
        "title": {
          "type": "text"
        }
      }
    }
  }
}


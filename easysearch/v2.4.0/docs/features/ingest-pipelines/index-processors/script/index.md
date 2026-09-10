---
title: "脚本处理器"
date: 0001-01-01
summary: "脚本处理器 #  script 处理器执行内联和存储的脚本，可以在数据导入过程中修改或转换 Easysearch 文档中的数据。由于脚本可能按文档重新编译，处理器使用脚本缓存以提高性能。有关在 Easysearch 中使用脚本的信息，请参阅脚本 API。
以下是为 script 处理器提供的语法：
{ &#34;processor&#34;: { &#34;script&#34;: { &#34;source&#34;: &#34;&lt;script_source&gt;&#34;, &#34;lang&#34;: &#34;&lt;script_language&gt;&#34;, &#34;params&#34;: { &#34;&lt;param_name&gt;&#34;: &#34;&lt;param_value&gt;&#34; } } } } 配置参数 #  下表列出了 script 处理器所需的和可选参数。
   参数 是否必填 描述     source 可选 要执行的 Painless 脚本。必须指定 id 或 source ，但不能同时指定两者。如果指定了 source ，则使用提供的源代码执行脚本。   id 可选 存储脚本的 ID，之前使用 Create Stored Script API 创建的。必须指定 id 或 source ，但不能同时指定两者。如果指定了 id ，则从具有指定 ID 的存储脚本中检索脚本源。   lang 可选 脚本的编程语言。默认为 painless 。   params 可选 可以传递给脚本的参数。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---



# 脚本处理器

`script` 处理器执行内联和存储的脚本，可以在数据导入过程中修改或转换 Easysearch 文档中的数据。由于脚本可能按文档重新编译，处理器使用脚本缓存以提高性能。有关在 Easysearch 中使用脚本的信息，请参阅脚本 API。


以下是为 `script` 处理器提供的语法：

```
{
  "processor": {
    "script": {
      "source": "<script_source>",
      "lang": "<script_language>",
      "params": {
        "<param_name>": "<param_value>"
      }
    }
  }
}
```

## 配置参数

下表列出了 `script` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                                                                |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `source`         | 可选     | 要执行的 Painless 脚本。必须指定 `id` 或 `source` ，但不能同时指定两者。如果指定了 `source` ，则使用提供的源代码执行脚本。                                          |
| `id`             | 可选     | 存储脚本的 ID，之前使用 Create Stored Script API 创建的。必须指定 `id` 或 `source` ，但不能同时指定两者。如果指定了 `id` ，则从具有指定 ID 的存储脚本中检索脚本源。 |
| `lang`           | 可选     | 脚本的编程语言。默认为 painless 。                                                                                                                                  |
| `params`         | 可选     | 可以传递给脚本的参数。                                                                                                                                              |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                                                              |
| `if`             | 可选     | 处理器运行的条件。                                                                                                                                                  |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                                   |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                                    |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                              |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `my-script-pipeline` 的管道，该管道使用 `script` 处理器将 `message` 字段转换为大写：

```
PUT _ingest/pipeline/my-script-pipeline
{
  "description": "Example pipeline using the ScriptProcessor",
  "processors": [
    {
      "script": {
        "source": "ctx.message = ctx.message.toUpperCase()",
        "lang": "painless",
        "description": "Convert message field to uppercase"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/my-script-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "hello, world!"
      }
    }
  ]
}
```

以下示例响应确认管道按预期工作：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "HELLO, WORLD!"
        },
        "_ingest": {
          "timestamp": "2024-05-30T16:24:23.30265405Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
POST testindex1/_doc/1?pipeline=my-script-pipeline
{
  "message": "hello, world!"
}
```

响应确认该文档已索引到 `testindex1` ，并且已将所有具有 `message` 字段的文档转换为大写：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 6,
  "_primary_term": 2
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```


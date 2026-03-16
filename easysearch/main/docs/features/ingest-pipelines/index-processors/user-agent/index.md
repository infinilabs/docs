---
title: "用户代理处理器"
date: 0001-01-01
summary: "用户代理处理器 #  user_agent 处理器用于从用户代理字符串中提取信息，例如客户端使用的浏览器、设备和操作系统。user_agent 处理器特别适用于分析用户行为，并根据用户设备、操作系统和浏览器识别趋势。它还可以帮助解决特定用户代理配置的问题。
以下是为 user_agent 处理器提供的语法：
{ &#34;processor&#34;: { &#34;user_agent&#34;: { &#34;field&#34;: &#34;user_agent&#34;, &#34;target_field&#34;: &#34;user_agent_info&#34; } } } 配置参数 #  下表列出了 user_agent 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含用户代理字符串的字段。   target_field 可选 存储提取的用户代理信息的字段。如果未指定，则信息存储在 user_agent 字段中。   ignore_missing 可选 指定处理器是否应忽略不包含指定 field 的文档。如果设置为 true ，则处理器在 field 不存在时不会修改文档。默认为 false 。   regex_file 可选 包含用于解析用户代理字符串的正则表达式模式的文件。此文件应位于 Easysearch 包中的 config/ingest-user-agent 目录下。如果未指定，则使用默认文件 regexes.yaml。   properties 可选 要从用户代理字符串中提取并添加到 target_field 的属性列表。如果未指定，则使用默认属性 name 、 major 、 minor 、 patch 、 build 、 os 、 os_name 、 os_major 、 os_minor 和 device 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 用户代理处理器

`user_agent` 处理器用于从用户代理字符串中提取信息，例如客户端使用的浏览器、设备和操作系统。`user_agent` 处理器特别适用于分析用户行为，并根据用户设备、操作系统和浏览器识别趋势。它还可以帮助解决特定用户代理配置的问题。


以下是为 `user_agent` 处理器提供的语法：

```
{
  "processor": {
    "user_agent": {
      "field": "user_agent",
      "target_field": "user_agent_info"
    }
  }
}
```

## 配置参数

下表列出了 `user_agent` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                                                                                                      |
| ---------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含用户代理字符串的字段。                                                                                                                                                                                |
| `target_field`   | 可选     | 存储提取的用户代理信息的字段。如果未指定，则信息存储在 `user_agent` 字段中。                                                                                                                              |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定 `field` 的文档。如果设置为 true ，则处理器在 `field` 不存在时不会修改文档。默认为 false 。                                                                                 |
| `regex_file`     | 可选     | 包含用于解析用户代理字符串的正则表达式模式的文件。此文件应位于 Easysearch 包中的 `config/ingest-user-agent` 目录下。如果未指定，则使用默认文件 `regexes.yaml`。                                           |
| `properties`     | 可选     | 要从用户代理字符串中提取并添加到 `target_field` 的属性列表。如果未指定，则使用默认属性 `name` 、 `major` 、 `minor` 、 `patch` 、 `build` 、 `os` 、 `os_name` 、 `os_major` 、 `os_minor` 和 `device` 。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                                                                                                    |
| `if`             | 可选     | 处理器运行的条件。                                                                                                                                                                                        |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                                                                         |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                                                                          |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                                                                    |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `user_agent_pipeline` 的管道，该管道使用 `user_agent` 处理器提取用户代理信息：

```
PUT _ingest/pipeline/user_agent_pipeline
{
  "description": "User agent pipeline",
  "processors": [
    {
      "user_agent": {
        "field": "user_agent",
        "target_field": "user_agent_info"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/user_agent_pipeline/_simulate
{
  "pipeline": "user_agent_pipeline",
  "docs": [
    {
      "_source": {
        "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3"
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
          "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3",
          "user_agent_info": {
            "name": "Chrome",
            "original": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3",
            "os": {
              "name": "Windows",
              "version": "10",
              "full": "Windows 10"
            },
            "device": {
              "name": "Other"
            },
            "version": "58.0.3029.110"
          }
        },
        "_ingest": {
          "timestamp": "2024-04-25T21:41:28.744407425Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=user_agent_pipeline
{
  "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
}
```

前一个请求将 `user_agent` 字符串解析为其组件，并将文档以及包含这些组件的所有文档索引到 `testindex1` 索引中，如下面的响应所示：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 66,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 65,
  "_primary_term": 47
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

响应包括原始的 `user_agent` 字段和包含设备、操作系统和浏览器信息的解析后的 `user_agent_info` 字段：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 66,
  "_seq_no": 65,
  "_primary_term": 47,
  "found": true,
  "_source": {
    "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36",
    "user_agent_info": {
      "original": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36",
      "os": {
        "name": "Mac OS X",
        "version": "10.15.7",
        "full": "Mac OS X 10.15.7"
      },
      "name": "Chrome",
      "device": {
        "name": "Mac"
      },
      "version": "90.0.4430.212"
    }
  }
}
```


---
title: "Grok 处理器"
date: 0001-01-01
summary: "Grok 处理器 #  grok 处理器用于通过模式匹配解析和结构化非结构化数据。您可以使用 grok 处理器从日志消息、Web 服务器访问日志、应用程序日志和其他遵循一致格式的日志数据中提取字段。
Grok 基础 #  grok 处理器使用一组预定义的模式来匹配输入文本的部分。每个模式由一个名称和一个正则表达式组成。例如，模式 %{IP:ip_address} 匹配 IP 地址并将其分配给字段 ip_address 。您可以将多个模式组合起来创建更复杂的表达式。例如，模式 %{IP:client} %{WORD:method} %{URIPATHPARM:request} %{NUMBER:bytes %NUMBER:duration} 匹配来自 Web 服务器访问日志的一行，并提取客户端 IP 地址、HTTP 方法、请求 URI、发送的字节数和请求持续时间。
 有关可用预定义模式的列表，请参阅 Grok 模式。
 grok 处理器基于 Oniguruma 正则表达式库构建，并支持该库中的所有模式。您可以使用 Grok 调试器工具测试和调试您的 grok 表达式。
 请注意，模式不是锚定的。为了性能和可靠性，请在您的模式中包含行首锚点（ ^ ）。
 语法 #  以下是为 grok 处理器的基本语法：
{ &#34;grok&#34;: { &#34;field&#34;: &#34;your_message&#34;, &#34;patterns&#34;: [&#34;your_patterns&#34;] } } 配置参数 #  要配置 grok 处理器，您有多种选项，允许您定义模式、匹配特定键和控制处理器的行为。下表列出了 grok 处理器的必选和可选参数。"
---


# Grok 处理器

`grok` 处理器用于通过模式匹配解析和结构化非结构化数据。您可以使用 `grok` 处理器从日志消息、Web 服务器访问日志、应用程序日志和其他遵循一致格式的日志数据中提取字段。


## Grok 基础

`grok` 处理器使用一组预定义的模式来匹配输入文本的部分。每个模式由一个名称和一个正则表达式组成。例如，模式 `%{IP:ip_address}` 匹配 IP 地址并将其分配给字段 `ip_address` 。您可以将多个模式组合起来创建更复杂的表达式。例如，模式 `%{IP:client} %{WORD:method} %{URIPATHPARM:request} %{NUMBER:bytes %NUMBER:duration}` 匹配来自 Web 服务器访问日志的一行，并提取客户端 IP 地址、HTTP 方法、请求 URI、发送的字节数和请求持续时间。

> 有关可用预定义模式的列表，请参阅 Grok 模式。

`grok` 处理器基于 Oniguruma 正则表达式库构建，并支持该库中的所有模式。您可以使用 Grok 调试器工具测试和调试您的 grok 表达式。

> 请注意，模式不是锚定的。为了性能和可靠性，请在您的模式中包含行首锚点（ `^` ）。

## 语法

以下是为 `grok` 处理器的基本语法：

```
{
  "grok": {
    "field": "your_message",
    "patterns": ["your_patterns"]
  }
}
```

## 配置参数

要配置 `grok` 处理器，您有多种选项，允许您定义模式、匹配特定键和控制处理器的行为。下表列出了 `grok` 处理器的必选和可选参数。

| 参数                  | 是否必填 | 描述                                                                                                                                                                                                                                                                                                                                           |
| --------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`               | 必填     | 包含要解析的文本的字段名称。                                                                                                                                                                                                                                                                                                                   |
| `patterns`            | 必填     | 用于匹配和提取命名捕获的 grok 表达式的列表。列表中第一个匹配的表达式被返回。                                                                                                                                                                                                                                                                   |
| `pattern_definitions` | 可选     | 一个用于定义当前处理器自定义模式的模式名称和模式元组的字典。如果模式与现有名称匹配，它将覆盖现有定义。                                                                                                                                                                                                                                         |
| `trace_match`         | 可选     | 当参数设置为 `true` 时，处理器向处理过的文档添加一个名为 `_grok_match_index` 的字段。该字段包含在 `patterns` 数组中成功匹配文档的模式索引。此信息对于调试和理解应用于文档的模式非常有用。默认为 `false` 。                                                                                                                                     |
| `capture_all_matches` | 可选     | 当设置为 `true` 时，捕获重复 `grok` 模式的所有匹配项，而不仅仅是第一个匹配项。例如，给定文本 `192.168.1.1 10.0.0.1 172.16.0.1` 和模式 `%{IP:ipAddress} %{IP:ipAddress} %{IP:ipAddress}` ，所有三个 IP 地址都收集到 `ipAddress` 字段中的数组中。仅与显式重复的模式一起工作，不与量化模式（如 `(%{IP:ipAddress})+` ）一起工作。默认为 `false` 。 |
| `description`         | 可选     | 处理器的一个简要描述。                                                                                                                                                                                                                                                                                                                         |
| `if`                  | 可选     | 处理器运行的条件。                                                                                                                                                                                                                                                                                                                             |
| `ignore_failure`      | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                                                                                                                                                                                                              |
| `ignore_missing`      | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。                                                                                                                                                                                                                      |
| `on_failure`          | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                                                                                                                                                                                                               |
| `tag`                 | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                                                                                                                                                                                                         |

## 如何使用

以下步骤指导您使用 grok 处理器创建一个摄取管道。

### 步骤 1：创建管道

以下查询创建了一个名为 `log_line` 的管道。它使用指定的模式从文档的 `message` 字段中提取字段。在这种情况下，它提取了 `clientip` 、 `timestamp` 和 `response_status` 字段：

```
PUT _ingest/pipeline/log_line
{
  "description": "Extract fields from a log line",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["^%{IPORHOST:clientip} %{HTTPDATE:timestamp} %{NUMBER:response_status:int}"]
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道。

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/log_line/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "198.126.12 10/Oct/2000:13:55:36 -0700 200"
      }
    }
  ]
}
```

以下响应确认管道按预期工作：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_type": "_doc",
        "_id": "_id",
        "_source": {
          "response_status": 200,
          "clientip": "198.126.12",
          "message": "198.126.12 10/Oct/2000:13:55:36 -0700 200",
          "timestamp": "10/Oct/2000:13:55:36 -0700"
        },
        "_ingest": {
          "timestamp": "2025-10-21T09:47:38.405544084Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=log_line
{
  "message": "127.0.0.1 198.126.12 10/Oct/2000:13:55:36 -0700 200"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

## 自定义模式

您可以使用默认模式，或者使用 `patterns_definitions` 参数将自定义模式添加到您的管道中。自定义 `Grok` 模式可以在管道中使用，以从不符合内置 `Grok` 模式的日志消息中提取结构化数据。这对于解析来自自定义应用程序的日志消息或解析以某种方式修改过的日志消息非常有用。自定义模式遵循简单的结构：每个模式都有一个唯一的名称以及定义其匹配行为的相应正则表达式。

以下是一个如何在配置中包含自定义模式的示例。在这个示例中，问题编号介于 `3` 到 `4` 位数字之间，并解析到 `issue_number` 字段，状态解析到 `status` 字段：

```
PUT _ingest/pipeline/log_line
{
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["^The issue number %{NUMBER:issue_number} is %{STATUS:status}"],
        "pattern_definitions" : {
          "NUMBER" : "\\d{3,4}",
          "STATUS" : "open|closed"
        }
      }
    }
  ]
}
```

## 如何追踪匹配模式

要追踪哪些模式匹配并填充了字段，您可以使用 `trace_match` 参数。以下是如何在您的配置中包含此参数的示例：

```
PUT _ingest/pipeline/log_line
{
  "description": "Extract fields from a log line",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["^%{HTTPDATE:timestamp} %{IPORHOST:clientip}", "%{IPORHOST:clientip} %{HTTPDATE:timestamp} %{NUMBER:response_status:int}"],
        "trace_match": true
      }
    }
  ]
}
```

当您模拟管道时，Easysearch 会返回包含 `grok_match_index` 的 `_ingest` 元数据，如下面的输出所示：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "127.0.0.1 198.126.12 10/Oct/2000:13:55:36 -0700 200",
          "response_status": 200,
          "clientip": "198.126.12",
          "timestamp": "10/Oct/2000:13:55:36 -0700"
        },
        "_ingest": {
          "_grok_match_index": "1",
          "timestamp": "2023-11-02T18:48:40.455619084Z"
        }
      }
    }
  ]
}
```


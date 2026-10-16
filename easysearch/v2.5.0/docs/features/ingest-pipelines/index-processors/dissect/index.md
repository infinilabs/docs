---
title: "解析处理器"
date: 0001-01-01
summary: "解析处理器 #  dissect 处理器从文档文本字段中提取值，并根据解析模式将它们映射到单个字段。该处理器非常适合从具有已知结构的日志消息中提取字段。与 grok 处理器不同，dissect 不使用正则表达式，语法更简单。
语法 #  以下是为 dissect 处理器提供的语法：
{ &#34;dissect&#34;: { &#34;field&#34;: &#34;source_field&#34;, &#34;pattern&#34;: &#34;%{dissect_pattern}&#34; } } 配置参数 #  下表列出了 dissect 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 要解析的数据所包含的字段名称。支持模板使用。   pattern 必填 用于从指定字段提取数据的 dissect 模式。   append_separator 可选 分隔字符或字符串，用于分隔附加字段。默认为 &quot;&quot; （空字符串）。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 解析处理器

`dissect` 处理器从文档文本字段中提取值，并根据解析模式将它们映射到单个字段。该处理器非常适合从具有已知结构的日志消息中提取字段。与 `grok` 处理器不同，`dissect` 不使用正则表达式，语法更简单。


## 语法

以下是为 `dissect` 处理器提供的语法：

```
{
  "dissect": {
    "field": "source_field",
    "pattern": "%{dissect_pattern}"
  }
}
```

## 配置参数

下表列出了 `dissect` 处理器所需的和可选参数。

| 参数               | 是否必填 | 描述                                                                                                                      |
| ------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `field`            | 必填     | 要解析的数据所包含的字段名称。支持模板使用。                                                                              |
| `pattern`          | 必填     | 用于从指定字段提取数据的 dissect 模式。                                                                                   |
| `append_separator` | 可选     | 分隔字符或字符串，用于分隔附加字段。默认为 `""` （空字符串）。                                                            |
| `description`      | 可选     | 处理器的一个简要描述。                                                                                                    |
| `if`               | 可选     | 处理器运行的条件。                                                                                                        |
| `ignore_failure`   | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                         |
| `ignore_missing`   | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。 |
| `on_failure`       | 可选     | 在处理器失败时运行的处理器列表。                                                                                          |
| `tag`              | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                    |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `dissect-test` 的管道，该管道使用 `dissect` 处理器解析日志行：

```
PUT /_ingest/pipeline/dissect-test
{
  "description": "Pipeline that dissects web server logs",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern": "%{client_ip} - - [%{timestamp}] \"%{http_method} %{url} %{http_version}\" %{response_code} %{response_size}"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/dissect-test/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "message": "192.168.1.10 - - [03/Nov/2023:15:20:45 +0000] \"POST /login HTTP/1.1\" 200 3456"
      }
    }
  ]
}
```

返回内容确认管道正在按预期工作：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "response_code": "200",
          "http_method": "POST",
          "http_version": "HTTP/1.1",
          "client_ip": "192.168.1.10",
          "message": """192.168.1.10 - - [03/Nov/2023:15:20:45 +0000] "POST /login HTTP/1.1" 200 3456""",
          "url": "/login",
          "response_size": "3456",
          "timestamp": "03/Nov/2023:15:20:45 +0000"
        },
        "_ingest": {
          "timestamp": "2023-11-03T22:28:32.830244044Z"
        }
      }
    }
  ]
}
```

### 步骤 3：获取文档

以下查询将文档提取到名为 @0# 的索引中：

```
PUT testindex1/_doc/1?pipeline=dissect-test
{
   "message": "192.168.1.10 - - [03/Nov/2023:15:20:45 +0000] \"POST /login HTTP/1.1\" 200 3456"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

## 解析匹配模式

解析模式是一种告诉 `dissect` 处理器如何将字符串解析为结构化格式的方法。该模式由要丢弃的字符串部分定义。例如， `%{client_ip} - - [%{timestamp}]` 解析模式将字符串 `"192.168.1.10 - - [03/Nov/2023:15:20:45 +0000] \"POST /login HTTP/1.1\" 200 3456"` 解析为以下字段：

```
client_ip: "192.168.1.1"
@timestamp: "03/Nov/2023:15:20:45 +0000"
```

解析模式的工作原理是将字符串与一组规则进行匹配。例如，第一条规则会丢弃一个空格。 `dissect` 处理器会找到这个空格，然后将 `client_ip` 的值赋给该空格之前的所有字符。下一条规则匹配 `[` 和 `]` 字符，然后将 `@timestamp` 的值赋给其间的所有内容。

### 建立解析匹配模式

构建解析模式时，务必注意要丢弃的字符串部分。如果丢弃过多的字符串， `dissect` 处理器可能无法成功解析剩余数据。相反，如果丢弃的字符串部分不足，处理器可能会创建不必要的字段。

如果模式中定义的任何 `%{keyname}` 没有值，则会引发异常。您可以通过在 `on_failure` 参数中提供错误处理步骤来处理此异常。

### 空值和取名跳过键

空键 `%{}` 或命名跳过键可用于匹配值，但会将该值排除在最终文档之外。如果您想要解析字符串但不需要存储其所有部分，这会很有用。

### 将匹配的值转换为非字符串数据类型

默认情况下，所有匹配的值都表示为字符串数据类型。如果需要将值转换为其他数据类型，可以使用 `convert` 处理器 。

## 修饰符

`dissect` 处理器支持可更改默认处理器行为的键修饰符。这些修饰符始终位于 `%{keyname}` 的左侧或右侧，并始终包含在 `%{}` 中。例如， `%{+keyname->}` 修饰符包含追加修饰符和右填充修饰符。键修饰符可用于将多个字段合并为一行输出、创建格式化的数据项列表或聚合来自多个来源的值。

下表列出了 `dissect` 处理器的主要修饰符。

| 修改符      | 名称       | 位置       | 用例                          | 描述                                                                                                                                                                   |
| ----------- | ---------- | ---------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `->`        | 跳过右部分 | 右         | `%{keyname->}`                | 告诉 `dissect` 处理器跳过右侧的任何重复字符。例如，可以使用 `%{timestamp->}` 来告诉处理器跳过 `timestamp` 后面的任何填充字符，例如两个连续的空格或任何不同的字符填充。 |
| `+`         | 附加       | 左边       | `%{keyname} %{+keyname}`      | 附加两个或多个字段。                                                                                                                                                   |
| `+` 和 `/n` | 按顺序附加 | 左右都可以 | `%{+keyname}/2 %{+keyname/1}` | 按指定顺序附加两个或多个字段。                                                                                                                                         |
| `?`         | 命名跳过键 | 左边       | `%{?skipme}`                  | 跳过输出中匹配的值。行为与 `%{}` 相同。                                                                                                                                |
| `*` 和 `&`  | 参考键     | 左边       | `%{*r1} %{&r1}`               | 将输出键设置为 `*` 的值，将输出值设置为 `&` 。                                                                                                                         |

以下部分提供了每个键修饰符的详细描述以及使用示例。

### 右填充修饰符（ `->` ）

解析算法非常精确，要求模式中的每个字符都与源字符串完全匹配。例如，模式 `%{hellokey} %{worldkey}` （一个空格）将匹配字符串“Hello world”（一个空格），但不会匹配字符串“Hello world”（两个空格），因为模式中只有一个空格，而源字符串有两个空格。

可以使用右填充修饰符来解决这个问题。当添加到模式 `%{helloworldkey->} %{worldkey}` 时，右填充修饰符将匹配 `Hello world` （1 个空格）、 `Hello world` （2 个空格），甚至 `Hello world` （10 个空格）。

右填充修饰符用于允许 `%{keyname->}` 后面的字符重复。右填充修饰符可以与任何其他修饰符一起应用于任何键。它应始终是最右边的修饰符，例如 `%{+keyname/1->}` 或 `%{}` 。

以下是如何使用右填充修饰符的示例：

```
%{city->}, %{state} %{zip}
```

在此模式中，右侧填充修饰符 `->` 应用于 `%{city}` 键。两个地址包含相同的信息，但第二个条目在 `city` 字段中多了一个单词 `City` 。右侧填充修饰符允许模式匹配这两个地址条目，即使它们的格式略有不同：

```
New York, NY 10017
New York City, NY 10017
```

以下示例管道使用带有空键 `%{->}` 右填充修饰符：

```
PUT /_ingest/pipeline/dissect-test
{
  "description": "Pipeline that dissects web server logs",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern": "[%{client_ip}]%{->}[%{timestamp}]"
      }
    }
  ]
}
```

您可以使用以下示例管道测试管道：

```
POST _ingest/pipeline/dissect-test/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "message": "[192.168.1.10]   [03/Nov/2023:15:20:45 +0000]"
      }
    }
  ]
}
```

返回的应类似于以下内容：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "client_ip": "192.168.1.10",
          "message": "[192.168.1.10]   [03/Nov/2023:15:20:45 +0000]",
          "timestamp": "03/Nov/2023:15:20:45 +0000"
        },
        "_ingest": {
          "timestamp": "2024-01-22T22:55:42.090569297Z"
        }
      }
    }
  ]
}
```

### 附加修饰符 ( `+` )

附加修饰符将两个或多个值合并为一个输出值。这些值按从左到右的顺序附加。您还可以指定在值之间插入可选分隔符。

以下是带有附加修饰符的示例管道：

```
PUT /_ingest/pipeline/dissect-test
{
  "description": "Pipeline that dissects web server logs",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern": "%{+address}, %{+address} %{+address}",
        "append_separator": "|"
      }
    }
  ]
}
```

您可以使用以下示例管道测试管道：

```
POST _ingest/pipeline/dissect-test/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "message": "New York, NY 10017"
      }
    }
  ]
}
```

子字符串附加到 `address` 字段，如以下响应所示：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "address": "New York|NY|10017",
          "message": "New York, NY 10017"
        },
        "_ingest": {
          "timestamp": "2024-01-22T22:30:54.516284637Z"
        }
      }
    }
  ]
}
```

### 附加顺序修饰符（ `+` 和 `/n` ）

带顺序附加修饰符会根据 `/` 后指定的顺序将两个或多个键的值合并为一个输出值。您可以灵活地自定义用于分隔附加值的分隔符。附加修饰符适用于将多个字段编译成单个格式化的输出行、构建结构化的数据项列表以及合并来自不同来源的值。

以下示例管道使用附加顺序修饰符来反转前一个管道中定义的模式顺序。此管道指定要在附加字段之间插入的分隔符。如果不指定分隔符，则所有值都将以不带分隔符的方式附加。

```
PUT /_ingest/pipeline/dissect-test
{
  "description": "Pipeline that dissects web server logs",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern": "%{+address/3}, %{+address/2} %{+address/1}",
        "append_separator": "|"
      }
    }
  ]
}
```

您可以使用以下示例管道测试管道：

```
POST _ingest/pipeline/dissect-test/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "message": "New York, NY 10017"
      }
    }
  ]
}
```

子字符串以相反的顺序附加到 `address` 字段，如以下响应所示：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "address": "10017|NY|New York",
          "message": "New York, NY 10017"
        },
        "_ingest": {
          "timestamp": "2024-01-22T22:38:24.305974178Z"
        }
      }
    }
  ]
}
```

### 命名跳过键修饰符

命名跳过键修饰符通过在模式中使用空键 `{}` 或 `?` 修饰符，从最终输出中排除特定匹配项。例如，以下模式是等效的： `%{firstName} %{lastName} %{?ignore} 和 %{firstName} %{lastName} %{}` 。命名跳过键修饰符可用于从输出中排除不相关或不必要的字段。

以下模式使用命名的跳过键从输出中排除某个字段（在本例中为 `ignore` ）。您可以为空键分配一个描述性名称（例如 `%{?ignore}` ，以明确说明应从最终输出中排除相应的值：

```
PUT /_ingest/pipeline/dissect-test
{
  "description": "Pipeline that dissects web server logs",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern": "%{firstName} %{lastName} %{?ignore}"
      }
    }
  ]
}
```

您可以使用以下示例管道测试管道：

```
POST _ingest/pipeline/dissect-test/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "message": "John Doe M.D."
      }
    }
  ]
}
```

返回的应类似于以下内容：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "firstName": "John",
          "lastName": "Doe",
          "message": "John Doe M.D."
        },
        "_ingest": {
          "timestamp": "2024-01-22T22:41:58.161475555Z"
        }
      }
    }
  ]
}
```

### 参考键（ `*` 和 `&` ）

引用键使用解析后的值作为结构化内容的键值对。这在处理以键值对形式部分记录数据的系统时非常有用。通过使用引用键，您可以保留键值关系并维护提取信息的完整性。

以下模式使用引用键将数据提取为结构化格式。在此示例中，提取了 `client_ip` 和两个键值对以获取以下值：

```
PUT /_ingest/pipeline/dissect-test
{
  "description": "Pipeline that dissects web server logs",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern": "%{client_ip} %{*a}:%{&a} %{*b}:%{&b}"
      }
    }
  ]
}
```

您可以使用以下示例管道测试管道：

```
POST _ingest/pipeline/dissect-test/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "message": "192.168.1.10 response_code:200 response_size:3456"
      }
    }
  ]
}
```

这两个键值对被提取到字段中，如以下响应所示：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "client_ip": "192.168.1.10",
          "response_code": "200",
          "message": "192.168.1.10 response_code:200 response_size:3456",
          "response_size": "3456"
        },
        "_ingest": {
          "timestamp": "2024-01-22T22:48:51.475535635Z"
        }
      }
    }
  ]
}
```


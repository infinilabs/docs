---
title: "分隔符负载分词过滤器（Delimited Payload）"
date: 0001-01-01
summary: "Delimited Payload 分词过滤器 #  delimited_payload 分词过滤器用于在分词过程中解析包含负载的词元。例如，字符串 red|1.5 fast|2.0 car|1.0 会被解析为词元 red（负载为 1.5）、fast（负载为 2.0）和 car（负载为 1.0）。当你的词元包含额外的关联数据（如权重、分数或其他数值），且你打算将这些数据用于评分或自定义查询逻辑时，这个过滤器就特别有用。该过滤器可以处理不同类型的负载，包括整数、浮点数和字符串，并将负载（额外的元数据）附加到词元上。
在文本分词时，delimited_payload 分词过滤器会解析每个词元，提取负载并将其附加到词元上。后续在查询中可以使用这个负载来影响评分、提升权重或实现其他自定义行为。
负载以 Base64 编码的字符串形式存储。默认情况下，负载不会随词元一起在查询响应中返回。若要返回负载，你必须配置额外的参数。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  分隔式负载分词过滤器有两个参数：
   参数 必需/可选 数据类型 描述     encoding 可选 字符串 指定附加到词元的负载的数据类型。这决定了在分析和查询期间如何解释负载数据。
有效值为：
- float：使用 IEEE 754 格式将负载解释为 32 位浮点数（例如，car\|2.5 中的 2.5）。
- identity：将负载解释为字符序列（例如，在 user\|admin 中，admin 被解释为字符串）。
- int：将负载解释为 32 位整数（例如，priority \| 1中的1）。"
---


# Delimited Payload 分词过滤器

`delimited_payload` 分词过滤器用于在分词过程中解析包含负载的词元。例如，字符串 `red|1.5 fast|2.0 car|1.0` 会被解析为词元 `red`（负载为 `1.5`）、`fast`（负载为 `2.0`）和 `car`（负载为 `1.0`）。当你的词元包含额外的关联数据（如权重、分数或其他数值），且你打算将这些数据用于评分或自定义查询逻辑时，这个过滤器就特别有用。该过滤器可以处理不同类型的负载，包括整数、浮点数和字符串，并将负载（额外的元数据）附加到词元上。

在文本分词时，`delimited_payload` 分词过滤器会解析每个词元，提取负载并将其附加到词元上。后续在查询中可以使用这个负载来影响评分、提升权重或实现其他自定义行为。

负载以 Base64 编码的字符串形式存储。默认情况下，负载不会随词元一起在查询响应中返回。若要返回负载，你必须配置额外的参数。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

分隔式负载分词过滤器有两个参数：

| 参数      | 必需/可选 | 数据类型 | 描述                                                                                                                                                                                                                                                                                                                                                                          |
| --------- | --------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| encoding  | 可选      | 字符串   | 指定附加到词元的负载的数据类型。这决定了在分析和查询期间如何解释负载数据。<br>有效值为：<br> - `float`：使用 IEEE 754 格式将负载解释为 32 位浮点数（例如，`car\|2.5` 中的 `2.5`）。<br> - `identity`：将负载解释为字符序列（例如，在 `user\|admin` 中，`admin` 被解释为字符串）。<br> - `int`：将负载解释为 32 位整数（例如，`priority \| 1`中的`1`）。<br>默认值为 `float`。 |
| delimiter | 可选      | 字符串   | 指定在输入文本中分隔词元及其负载的字符。默认值为竖线字符（`\|`）。                                                                                                                                                                                                                                                                                                            |

## 不带负载存储的分词示例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有分隔式负载过滤器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_payload_filter": {
          "type": "delimited_payload",
          "delimiter": "|",
          "encoding": "float"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "whitespace",
          "filter": ["my_payload_filter"]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "red|1.5 fast|2.0 car|1.0"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "red",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 0
    },
    {
      "token": "fast",
      "start_offset": 8,
      "end_offset": 16,
      "type": "word",
      "position": 1
    },
    {
      "token": "car",
      "start_offset": 17,
      "end_offset": 24,
      "type": "word",
      "position": 2
    }
  ]
}
```

## 带有负载存储的分词示例

若要在返回内容中带有负载，需创建一个存储词项向量的索引，并在索引映射中将 `term_vector` 设置为 `with_positions_payloads` 或 `with_positions_offsets_payloads`。例如，以下索引被配置为将负载存储到词项向量内容：

```
PUT /visible_payloads
{
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "term_vector": "with_positions_payloads",
        "analyzer": "custom_analyzer"
      }
    }
  },
  "settings": {
    "analysis": {
      "filter": {
        "my_payload_filter": {
          "type": "delimited_payload",
          "delimiter": "|",
          "encoding": "float"
        }
      },
      "analyzer": {
        "custom_analyzer": {
          "tokenizer": "whitespace",
          "filter": [ "my_payload_filter" ]
        }
      }
    }
  }
}
```

你可以使用以下请求将一个文档索引到这个索引中：

```
PUT /visible_payloads/_doc/1
{
  "text": "red|1.5 fast|2.0 car|1.0"
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
GET /visible_payloads/_termvectors/1
{
  "fields": ["text"]
}
```

返回内容中包含生成的词元，其中包括负载信息：

```
{
  "_index": "visible_payloads",
  "_type": "_doc",
  "_id": "1",
  "_version": 1,
  "found": true,
  "took": 5,
  "term_vectors": {
    "text": {
      "field_statistics": {
        "sum_doc_freq": 3,
        "doc_count": 1,
        "sum_ttf": 3
      },
      "terms": {
        "car": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 2,
              "payload": "P4AAAA=="
            }
          ]
        },
        "fast": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 1,
              "payload": "QAAAAA=="
            }
          ]
        },
        "red": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 0,
              "payload": "P8AAAA=="
            }
          ]
        }
      }
    }
  }
}
```


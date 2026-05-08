---
title: "正则表达式分词器（Pattern）"
date: 0001-01-01
summary: "Pattern 分词器 #  pattern 分词器是一种高度灵活的分词器，它允许你根据自定义的 Java 正则表达式将文本拆分为词元。与使用 Lucene 正则表达式的 simple_pattern 分词器和 simple_pattern_split 分词器不同，pattern 分词器能够处理更复杂、更细致的正则表达式模式，从而让你对文本的分词方式拥有更强的掌控力。
相关指南（先读这些） #    文本分析：识别词元  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用匹配词元生成器的分词器。该分词器会在 -、_ 或 . 字符处对文本进行分割。
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_pattern_tokenizer&#34;: { &#34;type&#34;: &#34;pattern&#34;, &#34;pattern&#34;: &#34;[-_.]&#34; } }, &#34;analyzer&#34;: { &#34;my_pattern_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;my_pattern_tokenizer&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_pattern_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Pattern 分词器

`pattern` 分词器是一种高度灵活的分词器，它允许你根据自定义的 Java 正则表达式将文本拆分为词元。与使用 Lucene 正则表达式的 `simple_pattern` 分词器和 `simple_pattern_split` 分词器不同，`pattern` 分词器能够处理更复杂、更细致的正则表达式模式，从而让你对文本的分词方式拥有更强的掌控力。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用匹配词元生成器的分词器。该分词器会在 `-`、`_` 或 `.` 字符处对文本进行分割。

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_pattern_tokenizer": {
          "type": "pattern",
          "pattern": "[-_.]"
        }
      },
      "analyzer": {
        "my_pattern_analyzer": {
          "type": "custom",
          "tokenizer": "my_pattern_tokenizer"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_pattern_analyzer"
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
  "analyzer": "my_pattern_analyzer",
  "text": "Easysearch-2024_v1.2"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Easysearch",
      "start_offset": 0,
      "end_offset": 10,
      "type": "word",
      "position": 0
    },
    {
      "token": "2024",
      "start_offset": 11,
      "end_offset": 15,
      "type": "word",
      "position": 1
    },
    {
      "token": "v1",
      "start_offset": 16,
      "end_offset": 18,
      "type": "word",
      "position": 2
    },
    {
      "token": "2",
      "start_offset": 19,
      "end_offset": 20,
      "type": "word",
      "position": 3
    }
  ]
}
```

## 参数说明

匹配词元生成器可以使用以下参数进行配置：

| 参数      | 必需/可选 | 数据类型 | 描述                                                                     |
| --------- | --------- | -------- | ------------------------------------------------------------------------ |
| `pattern` | 可选      | 字符串   | 用于将文本拆分为词元的模式，需使用 Java 正则表达式指定。默认值是 `\W+`。 |
| `flags`   | 可选      | 字符串   | 配置以竖线分隔的标志，用于应用于正则表达式。例如，`"CASE_INSENSITIVE \| MULTILINE \| DOTALL"`。 |
| `group`   | 可选      | 整数     | 指定用作词元的捕获组。默认值是 -1（在匹配处进行拆分）。                  |

## 使用分组参数

以下示例请求配置了一个 `group` 参数，该参数仅捕获第二个捕获组。

```
PUT /my_index_group2
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_pattern_tokenizer": {
          "type": "pattern",
          "pattern": "([a-zA-Z]+)(\\d+)",
          "group": 2
        }
      },
      "analyzer": {
        "my_pattern_analyzer": {
          "type": "custom",
          "tokenizer": "my_pattern_tokenizer"
        }
      }
    }
  }
}
```

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index_group2/_analyze
{
  "analyzer": "my_pattern_analyzer",
  "text": "abc123def456ghi"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "123",
      "start_offset": 3,
      "end_offset": 6,
      "type": "word",
      "position": 0
    },
    {
      "token": "456",
      "start_offset": 9,
      "end_offset": 12,
      "type": "word",
      "position": 1
    }
  ]
}
```


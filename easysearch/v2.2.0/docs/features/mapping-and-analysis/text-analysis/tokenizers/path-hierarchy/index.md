---
title: "路径层次分词器（Path Hierarchy）"
date: 0001-01-01
summary: "Path Hierarchy 分词器 #  path_hierarchy 分词器可以把文件路径（或类似的层次结构）的文本按照各个层级分解为词元。当处理诸如文件路径、网址或其他有分隔符的层次结构数据时，这个分词器特别有用。
相关指南（先读这些） #    文本分析：识别词元  文本分析基础  参考样例 #  以下示例请求创建一个名为 my_index 的新索引，并配置一个使用路径词元生成器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_path_tokenizer&#34;: { &#34;type&#34;: &#34;path_hierarchy&#34; } }, &#34;analyzer&#34;: { &#34;my_path_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;my_path_tokenizer&#34; } } } } } 生成的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my_index/_analyze { &#34;analyzer&#34;: &#34;my_path_analyzer&#34;, &#34;text&#34;: &#34;/users/john/documents/report.txt&#34; } 返回内容包含产生的词元
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;/users&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 6, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;/users/john&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 11, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;/users/john/documents&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 21, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;/users/john/documents/report."
---


# Path Hierarchy 分词器

`path_hierarchy` 分词器可以把文件路径（或类似的层次结构）的文本按照各个层级分解为词元。当处理诸如文件路径、网址或其他有分隔符的层次结构数据时，这个分词器特别有用。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建一个名为 `my_index` 的新索引，并配置一个使用路径词元生成器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_path_tokenizer": {
          "type": "path_hierarchy"
        }
      },
      "analyzer": {
        "my_path_analyzer": {
          "type": "custom",
          "tokenizer": "my_path_tokenizer"
        }
      }
    }
  }
}
```

## 生成的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_path_analyzer",
  "text": "/users/john/documents/report.txt"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "/users",
      "start_offset": 0,
      "end_offset": 6,
      "type": "word",
      "position": 0
    },
    {
      "token": "/users/john",
      "start_offset": 0,
      "end_offset": 11,
      "type": "word",
      "position": 0
    },
    {
      "token": "/users/john/documents",
      "start_offset": 0,
      "end_offset": 21,
      "type": "word",
      "position": 0
    },
    {
      "token": "/users/john/documents/report.txt",
      "start_offset": 0,
      "end_offset": 32,
      "type": "word",
      "position": 0
    }
  ]
}
```

## 参数说明

路径词元生成器可以使用以下参数进行配置。

| 参数          | 必需/可选 | 数据类型 | 描述                                                   |
| ------------- | --------- | -------- | ------------------------------------------------------ |
| `delimiter`   | 可选      | 字符串   | 指定用于分隔路径组件的字符。默认值为 `/`。             |
| `replacement` | 可选      | 字符串   | 配置用于替换词元中分隔符的字符。默认值为 `/`。         |
| `buffer_size` | 可选      | 整数     | 指定缓冲区大小。默认值为 `1024`。                      |
| `reverse`     | 可选      | 布尔值   | 若为 `true`，则按逆序生成词元。默认值为 `false`。      |
| `skip`        | 可选      | 整数     | 指定分词时要跳过的初始词元（层级）数量。默认值为 `0`。 |

## 使用分隔符和替换参数

以下示例请求配置了自定义的分隔符（`delimiter`）和替换(`replacement`)参数：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_path_tokenizer": {
          "type": "path_hierarchy",
          "delimiter": "\\",
          "replacement": "/"
        }
      },
      "analyzer": {
        "my_path_analyzer": {
          "type": "custom",
          "tokenizer": "my_path_tokenizer"
        }
      }
    }
  }
}
```

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_path_analyzer",
  "text": "C:\\users\\john\\documents\\report.txt"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "C:",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 0
    },
    {
      "token": "C:/users",
      "start_offset": 0,
      "end_offset": 8,
      "type": "word",
      "position": 0
    },
    {
      "token": "C:/users/john",
      "start_offset": 0,
      "end_offset": 13,
      "type": "word",
      "position": 0
    },
    {
      "token": "C:/users/john/documents",
      "start_offset": 0,
      "end_offset": 23,
      "type": "word",
      "position": 0
    },
    {
      "token": "C:/users/john/documents/report.txt",
      "start_offset": 0,
      "end_offset": 34,
      "type": "word",
      "position": 0
    }
  ]
}
```


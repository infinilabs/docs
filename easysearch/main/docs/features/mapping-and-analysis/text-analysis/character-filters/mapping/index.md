---
title: "映射字符过滤器（Mapping）"
date: 0001-01-01
summary: "Mapping 字符过滤器 #  mapping 字符过滤器接受一个用于字符替换的键值对映射。每当该过滤器遇到与某个键匹配的字符串时，它就会用相应的值来替换这些字符。替换值可以是空字符串。
该过滤器采用贪婪匹配方式，这意味着会匹配最长的匹配结果。
在分词过程之前，需要进行特定文本替换的场景下，mapping 字符过滤器会很有帮助。
相关指南（先读这些） #    文本分析基础  文本分析：规范化  参考样例 #  以下请求配置了一个映射字符过滤器，该过滤器可将罗马数字（如 I、II 或 III）转换为对应的阿拉伯数字（1、2 和 3）：
GET /_analyze { &#34;tokenizer&#34;: &#34;keyword&#34;, &#34;char_filter&#34;: [ { &#34;type&#34;: &#34;mapping&#34;, &#34;mappings&#34;: [ &#34;I =&gt; 1&#34;, &#34;II =&gt; 2&#34;, &#34;III =&gt; 3&#34;, &#34;IV =&gt; 4&#34;, &#34;V =&gt; 5&#34; ] } ], &#34;text&#34;: &#34;I have III apples and IV oranges&#34; } 返回内容中包含一个词元，其中罗马数字已被替换为阿拉伯数字：
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;1 have 3 apples and 4 oranges&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 32, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 0 } ] } 参数说明 #  你可以使用以下任意一个参数来配置键值映射。"
---


# Mapping 字符过滤器

`mapping` 字符过滤器接受一个用于字符替换的键值对映射。每当该过滤器遇到与某个键匹配的字符串时，它就会用相应的值来替换这些字符。替换值可以是空字符串。

该过滤器采用贪婪匹配方式，这意味着会匹配最长的匹配结果。

在分词过程之前，需要进行特定文本替换的场景下，`mapping` 字符过滤器会很有帮助。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参考样例

以下请求配置了一个映射字符过滤器，该过滤器可将罗马数字（如 I、II 或 III）转换为对应的阿拉伯数字（1、2 和 3）：

```
GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    {
      "type": "mapping",
      "mappings": [
        "I => 1",
        "II => 2",
        "III => 3",
        "IV => 4",
        "V => 5"
      ]
    }
  ],
  "text": "I have III apples and IV oranges"
}
```

返回内容中包含一个词元，其中罗马数字已被替换为阿拉伯数字：

```
{
  "tokens": [
    {
      "token": "1 have 3 apples and 4 oranges",
      "start_offset": 0,
      "end_offset": 32,
      "type": "word",
      "position": 0
    }
  ]
}
```

## 参数说明

你可以使用以下任意一个参数来配置键值映射。

| 参数            | 必需/可选 | 数据类型 | 描述                                                                                                                                                          |
| --------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mappings`      | 可选      | 数组     | 格式为 `key => value` 的键值对数组。在输入文本中找到的每个键都将被其对应的值替换。                                                                            |
| `mappings_path` | 可选      | 字符串   | 包含键值映射的 UTF-8 编码文件的路径。每个映射应在新的一行中以 `key => value` 的格式呈现。该路径可以是绝对路径，也可以是相对于 Easysearch 配置目录的相对路径。 |

### 使用自定义映射字符过滤器

你可以通过定义自己的映射集来创建自定义映射字符过滤器。以下请求将创建一个自定义字符过滤器，用于替换文本中的常见缩写：

```
PUT /test-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_abbr_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "custom_abbr_filter"
          ]
        }
      },
      "char_filter": {
        "custom_abbr_filter": {
          "type": "mapping",
          "mappings": [
            "BTW => By the way",
            "IDK => I don't know",
            "FYI => For your information"
          ]
        }
      }
    }
  }
}
```

使用以下请求来检查使用该分词器生成的词元：

```
GET /test-index/_analyze
{
  "tokenizer": "keyword",
  "char_filter": [ "custom_abbr_filter" ],
  "text": "FYI, updates to the workout schedule are posted. IDK when it takes effect, but we have some details. BTW, the finalized schedule will be released Monday."
}
```

返回内容显示这些缩写已被替换：

```
{
  "tokens": [
    {
      "token": "For your information, updates to the workout schedule are posted. I don't know when it takes effect, but we have some details. By the way, the finalized schedule will be released Monday.",
      "start_offset": 0,
      "end_offset": 153,
      "type": "word",
      "position": 0
    }
  ]
}
```


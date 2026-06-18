---
title: "简单模式分割分词器（Simple Pattern Split）"
date: 0001-01-01
summary: "Simple Pattern Split 分词器 #  simple_pattern_split 分词器使用正则表达式将文本分割成词元。该正则表达式定义了用于确定文本分割位置的模式。文本中任何匹配该模式的部分都会被用作分隔符，分隔符之间的文本则成为一个词元。当你想定义分隔符，并根据某个模式对文本的其余部分进行分词时，可以使用此分词器。
该分词器仅将输入文本中匹配到的部分（基于正则表达式）用作分隔符或边界，以将文本分割成词条。匹配到的部分不会包含在生成的词元中。例如，如果将分词器配置为在点号（.）处分割文本，输入文本为 one.two.three，那么生成的词元就是 one、two 和 three。点号本身不会包含在生成的词条中。
相关指南（先读这些） #    文本分析：识别词元  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用简单分割匹配词元生成器的分词器。该分词器被配置为在连字符（-）处分割文本：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_pattern_split_tokenizer&#34;: { &#34;type&#34;: &#34;simple_pattern_split&#34;, &#34;pattern&#34;: &#34;-&#34; } }, &#34;analyzer&#34;: { &#34;my_pattern_split_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;my_pattern_split_tokenizer&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_pattern_split_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Simple Pattern Split 分词器

`simple_pattern_split` 分词器使用正则表达式将文本分割成词元。该正则表达式定义了用于确定文本分割位置的模式。文本中任何匹配该模式的部分都会被用作分隔符，分隔符之间的文本则成为一个词元。当你想定义分隔符，并根据某个模式对文本的其余部分进行分词时，可以使用此分词器。

该分词器仅将输入文本中匹配到的部分（基于正则表达式）用作分隔符或边界，以将文本分割成词条。匹配到的部分不会包含在生成的词元中。例如，如果将分词器配置为在点号（`.`）处分割文本，输入文本为 `one.two.three`，那么生成的词元就是 `one`、`two` 和 `three`。点号本身不会包含在生成的词条中。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用简单分割匹配词元生成器的分词器。该分词器被配置为在连字符（`-`）处分割文本：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_pattern_split_tokenizer": {
          "type": "simple_pattern_split",
          "pattern": "-"
        }
      },
      "analyzer": {
        "my_pattern_split_analyzer": {
          "type": "custom",
          "tokenizer": "my_pattern_split_tokenizer"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_pattern_split_analyzer"
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
  "analyzer": "my_pattern_split_analyzer",
  "text": "2024-10-09"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "2024",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 0
    },
    {
      "token": "10",
      "start_offset": 5,
      "end_offset": 7,
      "type": "word",
      "position": 1
    },
    {
      "token": "09",
      "start_offset": 8,
      "end_offset": 10,
      "type": "word",
      "position": 2
    }
  ]
}
```

## 参数说明

简单分割匹配词元生成器可以使用以下参数进行配置。

| 参数               | 必需/可选 | 数据类型 | 描述                                                                                                           |
| ------------------ | --------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `max_token_length` | 可选      | 整数     | 设置生成词元的最大长度。若超过该长度，词元将按照 `max_token_length` 所配置的长度拆分为多个词元。默认值为 255。 |


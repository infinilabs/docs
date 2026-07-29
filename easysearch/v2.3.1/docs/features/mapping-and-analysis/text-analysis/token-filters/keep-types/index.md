---
title: "保留类型分词过滤器（Keep Types）"
date: 0001-01-01
summary: "Keep Types 分词过滤器 #  keep_types 分词过滤器是一种用于文本分析的分词过滤器，它可用于控制保留或丢弃哪些词元类型。不同的分词器会产生不同的词元类型，例如 &lt;HOST&gt;、&lt;NUM&gt; 或 &lt;ALPHANUM&gt;。
 注意：keyword 分词器、simple_pattern 分词器和 simple_pattern_split 分词器不支持 keep_types 分词过滤器，因为这些分词器不支持词元类型属性。
 相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  保留类型分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     types 必需 字符串列表 要保留或丢弃的词元类型列表（由 mode 参数决定）。   mode 可选 字符串 是要包含(include)还是排除(exclude) types 中指定的词元类型。默认值为 include（包含）。    参考样例 #  以下示例请求创建了一个名为 test_index 的新索引，并配置了一个带有保留类型过滤器的分析器：
PUT /test_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;custom_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;keep_types_filter&#34;] } }, &#34;filter&#34;: { &#34;keep_types_filter&#34;: { &#34;type&#34;: &#34;keep_types&#34;, &#34;types&#34;: [&#34;&lt;ALPHANUM&gt;&#34;] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Keep Types 分词过滤器

`keep_types` 分词过滤器是一种用于文本分析的分词过滤器，它可用于控制保留或丢弃哪些词元类型。不同的分词器会产生不同的词元类型，例如 `<HOST>`、`<NUM>` 或 `<ALPHANUM>`。

> **注意**：`keyword` 分词器、`simple_pattern` 分词器和 `simple_pattern_split` 分词器不支持 `keep_types` 分词过滤器，因为这些分词器不支持词元类型属性。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

保留类型分词过滤器可以使用以下参数进行配置。

| 参数    | 必需/可选 | 数据类型   | 描述                                                                                          |
| ------- | --------- | ---------- | --------------------------------------------------------------------------------------------- |
| `types` | 必需      | 字符串列表 | 要保留或丢弃的词元类型列表（由 `mode` 参数决定）。                                            |
| `mode`  | 可选      | 字符串     | 是要包含(`include`)还是排除(`exclude`) `types` 中指定的词元类型。默认值为 `include`（包含）。 |

## 参考样例

以下示例请求创建了一个名为 `test_index` 的新索引，并配置了一个带有保留类型过滤器的分析器：

```
PUT /test_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "keep_types_filter"]
        }
      },
      "filter": {
        "keep_types_filter": {
          "type": "keep_types",
          "types": ["<ALPHANUM>"]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
GET /test_index/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "Hello 2 world! This is an example."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "hello",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "world",
      "start_offset": 8,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "this",
      "start_offset": 15,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "is",
      "start_offset": 20,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "an",
      "start_offset": 23,
      "end_offset": 25,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "example",
      "start_offset": 26,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 6
    }
  ]
}
```


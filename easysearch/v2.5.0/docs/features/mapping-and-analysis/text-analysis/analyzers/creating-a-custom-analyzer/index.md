---
title: "创建自定义分析器（Custom）"
date: 0001-01-01
summary: "创建一个自定义分词器 #  要创建一个自定义分词器，需要指定以下组成内容：
 字符过滤器（零个或多个） 词元生成器（一个） 词元过滤器（零个或多个）  相关配置 #  以下参数可用于配置自定义分词器。
   参数 必填/可选 描述     type 可选 分词器类型。默认值为custom。你也可以给这个参数指定一个预构建的分词器。   tokenizer 必填 每个分词器必须要有一个词元生成器。   char_filter 可选 要包含在分词器中的字符过滤器列表。   filter 可选 要包含在分词器中的分词过滤器列表。   position_increment_gap 可选 在对包含多个词项内容的文本字段建立索引时，应用于各词项之间的额外间隔。有关更多信息，请参阅位置间隔增量。默认值为 100。    参考样例 #  以下示例展示了各种自定义分词器的配置。
自定义分词器用于去除 HTML 格式标签 #  以下示例中的分词器在分词之前会从文本中移除 HTML 格式标签：
PUT simple_html_strip_analyzer_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;html_strip_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;char_filter&#34;: [&#34;html_strip&#34;], &#34;tokenizer&#34;: &#34;whitespace&#34;, &#34;filter&#34;: [&#34;lowercase&#34;] } } } } } 使用以下请求来查看使用该分词器生成的词元："
---


# 创建一个自定义分词器

要创建一个自定义分词器，需要指定以下组成内容：

- 字符过滤器（零个或多个）
- 词元生成器（一个）
- 词元过滤器（零个或多个）

## 相关配置

以下参数可用于配置自定义分词器。

| 参数                   | 必填/可选 | 描述                                                                                                                       |
| ---------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------- |
| type                   | 可选      | 分词器类型。默认值为`custom`。你也可以给这个参数指定一个预构建的分词器。                                                   |
| tokenizer              | 必填      | 每个分词器必须要有一个词元生成器。                                                                                         |
| char_filter            | 可选      | 要包含在分词器中的字符过滤器列表。                                                                                         |
| filter                 | 可选      | 要包含在分词器中的分词过滤器列表。                                                                                         |
| position_increment_gap | 可选      | 在对包含多个词项内容的文本字段建立索引时，应用于各词项之间的额外间隔。有关更多信息，请参阅**位置间隔增量**。默认值为 100。 |

## 参考样例

以下示例展示了各种自定义分词器的配置。

### 自定义分词器用于去除 HTML 格式标签

以下示例中的分词器在分词之前会从文本中移除 HTML 格式标签：

```
PUT simple_html_strip_analyzer_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "html_strip_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "whitespace",
          "filter": ["lowercase"]
        }
      }
    }
  }
}
```

使用以下请求来查看使用该分词器生成的词元：

```
GET simple_html_strip_analyzer_index/_analyze
{
  "analyzer": "html_strip_analyzer",
  "text": "<p>Easysearch is <strong>awesome</strong>!</p>"
}
```

返回内容中包含生成的词元：

```
{
  "tokens": [
    {
      "token": "easysearch",
      "start_offset": 3,
      "end_offset": 13,
      "type": "word",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 14,
      "end_offset": 16,
      "type": "word",
      "position": 1
    },
    {
      "token": "awesome!",
      "start_offset": 25,
      "end_offset": 42,
      "type": "word",
      "position": 2
    }
  ]
}
```

### 自定义分词器实现同义词替换和字符串映射

以下示例中的分词器可以在同义词过滤之前替换特定的字符：

```
PUT mapping_analyzer_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "synonym_mapping_analyzer": {
          "type": "custom",
          "char_filter": ["underscore_to_space"],
          "tokenizer": "standard",
          "filter": ["lowercase", "stop", "synonym_filter"]
        }
      },
      "char_filter": {
        "underscore_to_space": {
          "type": "mapping",
          "mappings": ["_ => ' '"]
        }
      },
      "filter": {
        "synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "quick, fast, speedy",
            "big, large, huge"
          ]
        }
      }
    }
  }
}
```

使用以下请求来查看使用该分词器生成的词元：

```
GET mapping_analyzer_index/_analyze
{
  "analyzer": "synonym_mapping_analyzer",
  "text": "The slow_green_turtle is very large"
}
```

返回内容中包含生成的词元：

```
{
  "tokens": [
    {"token": "slow","start_offset": 4,"end_offset": 8,"type": "<ALPHANUM>","position": 1},
    {"token": "green","start_offset": 9,"end_offset": 14,"type": "<ALPHANUM>","position": 2},
    {"token": "turtle","start_offset": 15,"end_offset": 21,"type": "<ALPHANUM>","position": 3},
    {"token": "very","start_offset": 25,"end_offset": 29,"type": "<ALPHANUM>","position": 5},
    {"token": "large","start_offset": 30,"end_offset": 35,"type": "<ALPHANUM>","position": 6},
    {"token": "big","start_offset": 30,"end_offset": 35,"type": "SYNONYM","position": 6},
    {"token": "huge","start_offset": 30,"end_offset": 35,"type": "SYNONYM","position": 6}
  ]
}
```

### 自定义分词器实现数字规范化

以下示例中的分词器会通过去除破折号和空格的方式来规范化电话号码，并对规范化后的文本应用边缘 edge n-gram 以支持部分匹配：

```
PUT advanced_pattern_replace_analyzer_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "phone_number_analyzer": {
          "type": "custom",
          "char_filter": ["phone_normalization"],
          "tokenizer": "standard",
          "filter": ["lowercase", "edge_ngram"]
        }
      },
      "char_filter": {
        "phone_normalization": {
          "type": "pattern_replace",
          "pattern": "[-\\s]",
          "replacement": ""
        }
      },
      "filter": {
        "edge_ngram": {
          "type": "edge_ngram",
          "min_gram": 3,
          "max_gram": 10
        }
      }
    }
  }
}
```

使用以下请求来查看使用该分词器生成的词元：

```
GET advanced_pattern_replace_analyzer_index/_analyze
{
  "analyzer": "phone_number_analyzer",
  "text": "123-456 7890"
}
```

返回内容中包含生成的词元：

```
{
  "tokens": [
    {"token": "123","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0},
    {"token": "1234","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0},
    {"token": "12345","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0},
    {"token": "123456","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0},
    {"token": "1234567","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0},
    {"token": "12345678","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0},
    {"token": "123456789","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0},
    {"token": "1234567890","start_offset": 0,"end_offset": 12,"type": "<NUM>","position": 0}
  ]
}
```

## 处理正则表达式模式中的特殊字符

当在分词器中使用自定义正则表达式模式时，要确保能正确处理特殊字符或非英文字符。默认情况下，Java 的正则表达式仅将 `[A-Za-z0-9_]` 视为单词字符（`\w`）。这在使用 `\w` 或 `\b`（用于匹配单词和非单词字符之间的边界）时可能会导致意外行为。
例如，以下分词器尝试使用模式 `(\b\p{L}+\b)` 来匹配各语种（`\p{L}`）的一个或多个字母字符：

```
PUT /buggy_custom_analyzer
{
  "settings": {
    "analysis": {
      "filter": {
        "capture_words": {
          "type": "pattern_capture",
          "patterns": [
            "(\\b\\p{L}+\\b)"
          ]
        }
      },
      "analyzer": {
        "filter_only_analyzer": {
          "type": "custom",
          "tokenizer": "keyword",
          "filter": [
            "capture_words"
          ]
        }
      }
    }
  }
}
```

然而，这个分词器会错误地将 `él-empezó-a-reír` 分词为 `l`、`empez`、`a` 和 `reír`，因为 `\b` 无法匹配带重音符号的字符与字符串开头或结尾之间的边界。

为了正确处理特殊字符，你需要在模式中添加 Unicode 大小写标志 `(?U)`：

```
PUT /fixed_custom_analyzer
{
  "settings": {
    "analysis": {
      "filter": {
        "capture_words": {
          "type": "pattern_capture",
          "patterns": [
            "(?U)(\\b\\p{L}+\\b)"
          ]
        }
      },
      "analyzer": {
        "filter_only_analyzer": {
          "type": "custom",
          "tokenizer": "keyword",
          "filter": [
            "capture_words"
          ]
        }
      }
    }
  }
}
```

## 位置间隔增量

`position_increment_gap`（位置间隔增量）参数用于在对多值字段（如数组）建立索引时，设置词项之间的位置间隔。除非有明确的允许，这个间隔参数可确保短语查询不会出现跨词项匹配。例如，默认间隔值为 100，表示不同数组条目中的词项之间相隔 100 个位置距离，从而在短语搜索中防止出现意外的匹配情况。你可以调整这个值，或者将其设置为 0，以便让短语能够跨越词项进行匹配。

下面的示例通过使用 `match_phrase`（匹配短语）查询来演示 `position_increment_gap` 的作用。

1. 在 `test-index` 索引中索引一个文档：

```
PUT test-index/_doc/1
  {
    "names": [ "Slow green", "turtle swims"]
  }
```

2. 使用短语匹配查询（`match_phrase`）来查询文档：

```
GET test-index/_search
 {
   "query": {
     "match_phrase": {
       "names": {
         "query": "green turtle"
       }
     }
   }
 }
```

返回内容是空的，这是因为 `green` 和 `turtle` 这两个词项之间的距离是 100（即默认的 `position_increment_gap` 值）。

3. 现在，使用带有 `slop` 参数的 `match_phrase` 查询来查询文档，我们把该参数值设置成大于 `position_increment_gap` 的 101：

```
GET test-index/_search
 {
   "query": {
     "match_phrase": {
       "names": {
         "query": "green turtle",
         "slop": 101
       }
     }
   }
 }
```

返回内容包含了需要匹配的文档

```
 {
   "took": 4,
   "timed_out": false,
   "_shards": {
     "total": 1,
     "successful": 1,
     "skipped": 0,
     "failed": 0
   },
   "hits": {
     "total": {
       "value": 1,
       "relation": "eq"
     },
     "max_score": 0.010358453,
     "hits": [
       {
         "_index": "test-index",
         "_id": "1",
         "_score": 0.010358453,
         "_source": {
           "names": [
             "Slow green",
             "turtle swims"
           ]
         }
       }
     ]
   }
 }
```


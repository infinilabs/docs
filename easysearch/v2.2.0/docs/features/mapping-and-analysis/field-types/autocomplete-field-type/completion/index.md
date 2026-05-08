---
title: "自动补全字段类型（Completion）"
date: 0001-01-01
summary: "Completion 自动补全字段类型 #  自动补全字段类型通过补全建议器提供自动补全功能。补全建议器是一个前缀建议器，所以它只匹配文本的开头部分。补全建议器会创建一个内存中的数据结构，这提供了更快的查找速度，但会导致内存使用增加。在使用此功能之前，你需要将所有可能的补全项上传到索引中。
代码样例 #  创建一个包含补全字段的映射：
PUT chess_store { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;suggestions&#34;: { &#34;type&#34;: &#34;completion&#34; }, &#34;product&#34;: { &#34;type&#34;: &#34;keyword&#34; } } } } 将建议内容索引到 Easysearch 中：
PUT chess_store/_doc/1 { &#34;suggestions&#34;: { &#34;input&#34;: [&#34;Books on openings&#34;, &#34;Books on endgames&#34;], &#34;weight&#34; : 10 } } 参数 #  下表列出了补全字段接受的参数。
   参数 描述     input 可能的补全项列表，可以是字符串或字符串数组。不能包含 \u0000 (null),\u001f (信息分隔符一) 或 \u001e (信息分隔符二)。必需。   weight 用于对建议进行排序的正整数或正整数字符串。可选。    可以按以下方式索引多个建议："
---

# Completion 自动补全字段类型

自动补全字段类型通过补全建议器提供自动补全功能。补全建议器是一个前缀建议器，所以它只匹配文本的开头部分。补全建议器会创建一个内存中的数据结构，这提供了更快的查找速度，但会导致内存使用增加。在使用此功能之前，你需要将所有可能的补全项上传到索引中。


## 代码样例

创建一个包含补全字段的映射：
```json
PUT chess_store
{
  "mappings": {
    "properties": {
      "suggestions": {
        "type": "completion"
      },
      "product": {
        "type": "keyword"
      }
    }
  }
}
```

将建议内容索引到 Easysearch 中：
```json
PUT chess_store/_doc/1
{
  "suggestions": {
      "input": ["Books on openings", "Books on endgames"],
      "weight" : 10
    }
}
```


## 参数

下表列出了补全字段接受的参数。

| 参数 | 描述 |
|------|------|
| `input` | 可能的补全项列表，可以是字符串或字符串数组。不能包含 `\u0000` (null),`\u001f` (信息分隔符一) 或 `\u001e` (信息分隔符二)。必需。 |
| `weight` | 用于对建议进行排序的正整数或正整数字符串。可选。 |

可以按以下方式索引多个建议：
```json
PUT chess_store/_doc/2
{
  "suggestions": [
    {
      "input": "Chess set",
      "weight": 20
    },
    {
      "input": "Chess pieces",
      "weight": 10
    },
    {
      "input": "Chess board",
      "weight": 5
    }
  ]
}
```

或者，你可以使用以下简写方式（注意在这种表示法中不能提供 `weight` 参数）：
```json
PUT chess_store/_doc/3
{
  "suggestions" : [ "Chess clock", "Chess timer" ]
}
```


## 查询补全字段类型

要查询补全字段类型，需要指定要搜索的前缀和要查找建议的字段名称。

查询以单词 "chess" 开头的内容建议：
```json
GET chess_store/_search
{
  "suggest": {
    "product-suggestions": {
      "prefix": "chess",        
      "completion": {         
          "field": "suggestions"
      }
    }
  }
}
```

返回内容包含自动补全建议：

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "product-suggestions" : [
      {
        "text" : "chess",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "Chess set",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "2",
            "_score" : 20.0,
            "_source" : {
              "suggestions" : [
                {
                  "input" : "Chess set",
                  "weight" : 20
                },
                {
                  "input" : "Chess pieces",
                  "weight" : 10
                },
                {
                  "input" : "Chess board",
                  "weight" : 5
                }
              ]
            }
          },
          {
            "text" : "Chess clock",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "3",
            "_score" : 1.0,
            "_source" : {
              "suggestions" : [
                "Chess clock",
                "Chess timer"
              ]
            }
          }
        ]
      }
    ]
  }
}
```

在返回内容中，`_score` 字段包含了在索引时设置的 `weight` 参数值。`text` 字段填充了建议的 `input` 参数。

默认情况下，响应包含整个文档，包括 `_source` 字段，这可能会影响性能。要只返回 `suggestions` 字段，你可以在 `_source` 参数中指定。你还可以通过指定 `size` 参数来限制返回的建议数量。
```json
GET chess_store/_search
{
  "_source": "suggestions", 
  "suggest": {
    "product-suggestions": {
      "prefix": "chess",        
      "completion": {         
          "field": "suggestions",
          "size" : 3
      }
    }
  }
}
```

返回内容会包含建议内容：
```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "product-suggestions" : [
      {
        "text" : "chess",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "Chess set",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "2",
            "_score" : 20.0,
            "_source" : {
              "suggestions" : [
                {
                  "input" : "Chess set",
                  "weight" : 20
                },
                {
                  "input" : "Chess pieces",
                  "weight" : 10
                },
                {
                  "input" : "Chess board",
                  "weight" : 5
                }
              ]
            }
          },
          {
            "text" : "Chess clock",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "3",
            "_score" : 1.0,
            "_source" : {
              "suggestions" : [
                "Chess clock",
                "Chess timer"
              ]
            }
          }
        ]
      }
    ]
  }
}
```
> 要利用源内容过滤功能，请在 `_search` API 上使用建议功能。`_suggest` API 不支持源内容过滤。

## 自动补全查询参数

下表列出了自动补全查询接受的参数。

| 参数 | 描述 |
|------|------|
| `field` | 一个字符串，指定要运行查询的字段。必需。 |
| `size` | 一个整数，指定返回的建议的最大数量。可选。默认值为 5。 |
| `skip_duplicates` | 一个布尔值，指定是否跳过重复的建议。可选。默认值为 `false`。 |

## 自动补全的模糊查询

要允许模糊匹配，可以为补全查询指定 `fuzziness` 参数。在这种情况下，即使用户输入错误的搜索词，自动补全查询仍然会返回结果。此外，匹配查询的前缀越长，文档的得分越高。
```json
GET chess_store/_search
{
  "suggest": {
    "product-suggestions": {
      "prefix": "chesc",        
      "completion": {         
          "field": "suggestions",
          "size" : 3,
          "fuzzy" : {
            "fuzziness" : "AUTO"
          }
      }
    }
  }
}
```

> 要使用所有默认的模糊选项，可以指定 `"fuzzy": {}` 或 `"fuzzy": true`。

下表列出了自动补全的模糊查询接受的参数。所有参数都是可选的。

| 参数 | 描述 |
|------|------| 
| `fuzziness` | 模糊度可以设置为以下之一：1. 一个整数，指定允许的最大 Levenshtein 距离。2. `AUTO`：0-2 个字符的字符串必须完全匹配，3-5 个字符的字符串允许 1 次编辑，超过 5 个字符的字符串允许 2 次编辑。默认值为 `AUTO。` | 
| `min_length` | 一个整数，指定输入的最小长度，以开始返回建议。如果搜索词的长度小于 `min_length`，则不返回建议。默认值为 3。 | 
| `prefix_length` | 一个整数，指定匹配的前缀的最小长度，以开始返回建议。如果 `prefix_length` 的前缀不匹配，但搜索词仍然在 Levenshtein 距离内，则不返回建议。默认值为 `1`。 | 
| `transpositions` | 一个布尔值，指定将相邻字符的交换（transpositions）计为一次编辑，而不是两次编辑。示例：建议的 `input` 参数是 `abcde`，`fuzziness` 是 1。如果 `transpositions` 设置为 `true`，则 `abdce` 匹配，但如果 `transpositions` 设置为 `false`，则 `abdce` 不匹配。默认值为 `true`。 | 
| `unicode_aware` | 一个布尔值，指定是否使用 Unicode 代码点来测量编辑距离、转置和长度。如果 `unicode_aware` 设置为 `true`，则测量速度较慢。默认值为 `false`，在这种情况下，距离以字节为单位测量。 |

## 正则表达式查询

可以使用正则表达式来定义自动补全查询的前缀。

例如，要搜索以 "a" 开头并且后面有 "d" 的字符串，可以使用以下查询：
```json
GET chess_store/_search
{
  "suggest": {
    "product-suggestions": {
      "regex": "a.*d",        
      "completion": {         
          "field": "suggestions"
      }
    }
  }
}
```

返回内容匹配字符串 "abcde"：
```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "product-suggestions" : [
      {
        "text" : "a.*d",
        "offset" : 0,
        "length" : 4,
        "options" : [
          {
            "text" : "abcde",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "2",
            "_score" : 20.0,
            "_source" : {
              "suggestions" : [
                {
                  "input" : "abcde",
                  "weight" : 20
                }
              ]
            }
          }
        ]
      }
    ]
  }
}
```


---
title: "边输入边搜索字段类型（Search-as-you-type）"
date: 0001-01-01
summary: "Search-as-you-type 字段类型 #  search-as-you-type 字段类型通过前缀和中缀补全提供边输入边搜索的功能。
代码样例 #  将字段映射为 search-as-you-type 类型时，会为该字段创建 n-gram 子字段，其中 n 的范围为 [2, max_shingle_size]。此外，还会创建一个索引前缀子字段。
创建一个 search-as-you-type 的映射字段
PUT books { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;suggestions&#34;: { &#34;type&#34;: &#34;search_as_you_type&#34; } } } } 除了创建 suggestions 字段外，还会生成 suggestions._2gram、suggestions._3gram 和 suggestions._index_prefix 字段。
以下是使用 search-as-you-type 字段索引文档的示例：
PUT books/_doc/1 { &#34;suggestions&#34;: &#34;one two three four&#34; } 要匹配任意顺序的词项，可以使用 bool_prefix 或 multi-match 查询。
这些查询会将搜索词项按顺序匹配的文档排名提高，而将词项顺序不一致的文档排名降低。
GET books/_search { &#34;query&#34;: { &#34;multi_match&#34;: { &#34;query&#34;: &#34;tw one&#34;, &#34;type&#34;: &#34;bool_prefix&#34;, &#34;fields&#34;: [ &#34;suggestions&#34;, &#34;suggestions."
---


# Search-as-you-type 字段类型

**search-as-you-type** 字段类型通过前缀和中缀补全提供边输入边搜索的功能。

## 代码样例

将字段映射为 **search-as-you-type** 类型时，会为该字段创建 n-gram 子字段，其中 n 的范围为 [2, max_shingle_size]。此外，还会创建一个索引前缀子字段。

创建一个 search-as-you-type 的映射字段

```
PUT books
{
  "mappings": {
    "properties": {
      "suggestions": {
        "type": "search_as_you_type"
      }
    }
  }
}
```

除了创建 suggestions 字段外，还会生成 suggestions.\_2gram、suggestions.\_3gram 和 suggestions.\_index_prefix 字段。

以下是使用 search-as-you-type 字段索引文档的示例：

```
PUT books/_doc/1
{
  "suggestions": "one two three four"
}
```

要匹配任意顺序的词项，可以使用 **bool_prefix** 或 **multi-match** 查询。

这些查询会将搜索词项按顺序匹配的文档排名提高，而将词项顺序不一致的文档排名降低。

```
GET books/_search
{
  "query": {
    "multi_match": {
      "query": "tw one",
      "type": "bool_prefix",
      "fields": [
        "suggestions",
        "suggestions._2gram",
        "suggestions._3gram"
      ]
    }
  }
}
```

返回内容包含匹配的文档：

```
{
  "took" : 13,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "suggestions" : "one two three four"
        }
      }
    ]
  }
}
```

要按顺序匹配词项，可以使用 match_phrase_prefix 查询：

```
GET books/_search
{
  "query": {
    "match_phrase_prefix": {
      "suggestions": "two th"
    }
  }
}
```

返回内容包含匹配到的文档：

```
{
  "took" : 23,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.4793051,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.4793051,
        "_source" : {
          "suggestions" : "one two three four"
        }
      }
    ]
  }
}
```

要精确匹配最后的词项，可以使用 **match_phrase** 查询：

```
GET books/_search
{
  "query": {
    "match_phrase": {
      "suggestions": "four"
    }
  }
}
```

返回内容：

```
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "suggestions" : "one two three four"
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 **search-as-you-type** 字段类型支持的参数，所有参数均为可选项。  
| 参数 | 描述 | 默认值 |  
|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|  
| **analyzer** | 指定该字段使用的分析器，默认情况下用于索引和搜索阶段。如果需要在搜索时覆盖分析器，请设置 **search_analyzer** 参数。默认使用 **standard analyzer**，基于 Unicode 文本分割算法和语法规则进行分词。此设置适用于根字段和子字段。 | **standard analyzer** |  
| **index** | 布尔值，指定字段是否可被搜索。此设置适用于根字段和子字段。 | **true** |  
| **index_options** | 指定索引中存储的信息，以支持搜索和高亮显示。可选值：**docs** (仅文档编号)、**freqs** (文档编号和词频)、**positions** (文档编号、词频和位置)、**offsets** (文档编号、词频、位置及字符偏移)。此设置适用于根字段和子字段。 | **positions** |  
| **max_shingle_size** | 整数值，指定最大 n-gram 大小，有效范围为 [2, 4]。创建的 n-gram 范围为 [2, max_shingle_size]。默认值为 3，会生成 2-gram 和 3-gram。较大的值更适合具体查询，但会导致索引大小增加。 | **3** |  
| **norms** | 布尔值，指定在计算相关性评分时是否使用字段长度。适用于根字段和 n-gram 子字段（默认为 **false**）。不适用于前缀子字段（在前缀子字段中默认为 **false**）。 | **false** |  
| **search_analyzer** | 指定搜索时使用的分析器。默认值为 **analyzer** 参数中指定的分析器。适用于根字段和子字段。 | **analyzer** 参数值 |  
| **search_quote_analyzer** | 指定搜索短语时使用的分析器。默认值为 **analyzer** 参数中指定的分析器。适用于根字段和子字段。 | **analyzer** 参数值 |  
| **similarity** | 用于计算相关性评分的排序算法。默认值为 **BM25**。适用于根字段和子字段。 | **BM25** |  
| **store** | 布尔值，指定字段值是否单独存储并可从 `_source` 外检索。仅适用于根字段。 | **false** |  
| **term_vector** | 布尔值，指定是否为该字段存储词向量。适用于根字段和 n-gram 子字段（默认为 **no**）。不适用于前缀子字段。 | **no** |


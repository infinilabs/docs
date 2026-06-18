---
title: "简繁转换分析器（STConvert）"
date: 0001-01-01
summary: "STConvert 分析器 #  stconvert 分析器可在索引与查询阶段将简体中文与繁体中文之间进行双向转换，解决两种文字体系混合检索的问题。
相关指南（先读这些） #    文本分析基础  文本分析：规范化  参数说明 #     参数 说明 默认值     convert_type 转换方向，可选：s2t（简 → 繁），t2s（繁 → 简） s2t   keep_both 是否同时保留转换前后两种 token false   delimiter 当 keep_both=true 时，两种 token 之间的分隔符 ,    使用介绍 #  映射创建
PUT /stconvert/ { &#34;settings&#34; : { &#34;analysis&#34; : { &#34;analyzer&#34; : { &#34;tsconvert&#34; : { &#34;tokenizer&#34; : &#34;tsconvert&#34; } }, &#34;tokenizer&#34; : { &#34;tsconvert&#34; : { &#34;type&#34; : &#34;stconvert&#34;, &#34;delimiter&#34; : &#34;#&#34;, &#34;keep_both&#34; : false, &#34;convert_type&#34; : &#34;t2s&#34; } }, &#34;filter&#34;: { &#34;tsconvert&#34; : { &#34;type&#34; : &#34;stconvert&#34;, &#34;delimiter&#34; : &#34;#&#34;, &#34;keep_both&#34; : false, &#34;convert_type&#34; : &#34;t2s&#34; } }, &#34;char_filter&#34; : { &#34;tsconvert&#34; : { &#34;type&#34; : &#34;stconvert&#34;, &#34;convert_type&#34; : &#34;t2s&#34; } } } } } 分词测试"
---


# STConvert 分析器

`stconvert` 分析器可在索引与查询阶段将**简体中文**与**繁体中文**之间进行双向转换，解决两种文字体系混合检索的问题。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

| 参数             | 说明                                                     | 默认值    |
| ---------------- | -------------------------------------------------------- | --------- |
| `convert_type` | 转换方向，可选：`s2t`（简 → 繁），`t2s`（繁 → 简） | `s2t`   |
| `keep_both`    | 是否同时保留转换前后两种 token                           | `false` |
| `delimiter`    | 当 `keep_both=true` 时，两种 token 之间的分隔符        | `,`     |

## 使用介绍

映射创建

```json
PUT /stconvert/
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "tsconvert" : {
                    "tokenizer" : "tsconvert"
                    }
            },
            "tokenizer" : {
                "tsconvert" : {
                    "type" : "stconvert",
                    "delimiter" : "#",
                    "keep_both" : false,
                    "convert_type" : "t2s"
                }
            },
             "filter": {
               "tsconvert" : {
                     "type" : "stconvert",
                     "delimiter" : "#",
                     "keep_both" : false,
                     "convert_type" : "t2s"
                 }
             },
            "char_filter" : {
                "tsconvert" : {
                    "type" : "stconvert",
                    "convert_type" : "t2s"
                }
            }
        }
    }
}
```

分词测试

```json
GET stconvert/_analyze
{
  "tokenizer" : "keyword",
  "filter" : ["lowercase"],
  "char_filter" : ["tsconvert"],
  "text" : "国际國際"
}

# 返回
{
  "tokens": [
    {
      "token": "国际国际",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 0
    }
  ]
}
```

归一化方法

```json
PUT index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "tsconvert": {
          "type": "stconvert",
          "convert_type": "t2s"
        }
      },
      "normalizer": {
        "my_normalizer": {
          "type": "custom",
          "char_filter": [
            "tsconvert"
          ],
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "foo": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      }
    }
  }
}

PUT index/_doc/1
{
  "foo": "國際"
}

PUT index/_doc/2
{
  "foo": "国际"
}

GET index/_search
{
  "query": {
    "term": {
      "foo": "国际"
    }
  }
}

GET index/_search
{
  "query": {
    "term": {
      "foo": "國際"
    }
  }
}
```


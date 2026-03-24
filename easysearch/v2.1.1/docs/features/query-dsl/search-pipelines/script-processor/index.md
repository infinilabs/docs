---
title: "脚本处理器"
date: 0001-01-01
summary: "脚本处理器（Script Processor） #  版本引入：1.14.0
script 查询重写处理器用于拦截搜索请求，并在请求中添加一个内联的 Painless 脚本，该脚本会在接收到请求时执行。脚本仅能操作以下请求字段：
 from size explain version seq_no_primary_term track_scores track_total_hits min_score terminate_after profile  请求体字段 #  下表列出了该处理器支持的所有配置字段。
   字段 数据类型 说明     source 内联脚本 要执行的脚本代码。必填。   lang 字符串 脚本语言。可选，默认为 painless，目前仅支持 painless。   tag 字符串 处理器的唯一标识符，用于调试或跟踪。可选。   description 字符串 对该处理器的描述信息。可选。   ignore_failure 布尔值 若为 true，当此处理器执行失败时，Easysearch 将忽略错误并继续执行后续处理器。可选，默认值为 false。    示例 #  以下请求创建一个名为 explain_one_result 的搜索管道，其中包含一个 script 查询重写处理器。该脚本的作用是：当请求返回多个结果时自动关闭 explain 功能，因为 explain 是一项开销较大的操作；仅在返回单个结果时启用。"
---


# 脚本处理器（Script Processor）
版本引入：1.14.0

`script` 查询重写处理器用于拦截搜索请求，并在请求中添加一个内联的 Painless 脚本，该脚本会在接收到请求时执行。脚本仅能操作以下请求字段：

- `from`
- `size`
- `explain`
- `version`
- `seq_no_primary_term`
- `track_scores`
- `track_total_hits`
- `min_score`
- `terminate_after`
- `profile`


## 请求体字段

下表列出了该处理器支持的所有配置字段。

| 字段 | 数据类型 | 说明                                                               |
| :--- | :--- |:-----------------------------------------------------------------|
| `source` | 内联脚本 | 要执行的脚本代码。**必填**。                                                 |
| `lang` | 字符串 | 脚本语言。可选，默认为 `painless`，目前仅支持 `painless`。                         |
| `tag` | 字符串 | 处理器的唯一标识符，用于调试或跟踪。可选。                                            |
| `description` | 字符串 | 对该处理器的描述信息。可选。                                                   |
| `ignore_failure` | 布尔值 | 若为 `true`，当此处理器执行失败时，Easysearch 将忽略错误并继续执行后续处理器。可选，默认值为 `false`。 |

## 示例

以下请求创建一个名为 `explain_one_result` 的搜索管道，其中包含一个 `script` 查询重写处理器。该脚本的作用是：当请求返回多个结果时自动关闭 `explain` 功能，因为 `explain` 是一项开销较大的操作；仅在返回单个结果时启用。

```auto
PUT /_search/pipeline/explain_one_result
{
  "description": "一个仅对单个结果启用 explain 操作的搜索管道",
  "rewrite_processors": [
    {
      "script": {
        "lang": "painless",
        "source": "if (ctx._source['size'] > 1) { ctx._source['explain'] = false } else { ctx._source['explain'] = true }"
      }
    }
  ]
}
```

### 说明

- `ctx._source` 表示当前搜索请求的请求体内容。
- 上述脚本逻辑为：如果 `size`（返回结果数量）大于 1，则设置 `"explain": false`，否则设置为 `true`。
- 这样可以避免在大量结果上执行高开销的评分解释（`explain`），从而提升性能。

### 使用搜索管道

在搜索请求中通过 `search_pipeline` 参数指定该管道：

```auto
GET /my_index/_search?search_pipeline=explain_one_result
{
  "query": {
    "match": {
      "message": "public"
    }
  },
  "size": 1
}
```



---
title: "忽略字段元数据字段（_ignored）"
date: 0001-01-01
summary: "_ignored 元数据字段 #  _ignored 字段帮助您管理文档中与格式错误数据相关的问题。由于在 索引映射中启用了 ignore_malformed 设置，此字段用于存储在数据索引过程中被忽略的字段名称。
_ignored 字段允许您搜索和识别包含被忽略字段的文档，以及被忽略的具体字段名称。这对于故障排除很有用。
您可以使用 term、terms 和 exists 查询来查询 _ignored 字段。
 注意：只有当索引映射中启用了 ignore_malformed 设置时，才会填充 _ignored 字段。如果 ignore_malformed 设置为 false（默认值），则格式错误的字段将导致整个文档被拒绝，并且不会填写 _ignored 字段。
 相关指南（先读这些） #    映射基础  元数据字段  以下示例展示了如何使用 _ignored 字段：
GET _search { &#34;query&#34;: { &#34;exists&#34;: { &#34;field&#34;: &#34;_ignored&#34; } } } 使用 _ignored 字段的索引请求示例 #  以下示例向 test-ignored 索引添加一个新文档，其中 ignore_malformed 设置为 true，这样在数据索引时不会抛出错误：
PUT test-ignored { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;length&#34;: { &#34;type&#34;: &#34;long&#34;, &#34;ignore_malformed&#34;: true } } } } POST test-ignored/_doc { &#34;title&#34;: &#34;correct text&#34;, &#34;length&#34;: &#34;not a number&#34; } GET test-ignored/_search { &#34;query&#34;: { &#34;exists&#34;: { &#34;field&#34;: &#34;_ignored&#34; } } } 示例返回内容 #  { &#34;took&#34;: 42, &#34;timed_out&#34;: false, &#34;_shards&#34;: { &#34;total&#34;: 1, &#34;successful&#34;: 1, &#34;skipped&#34;: 0, &#34;failed&#34;: 0 }, &#34;hits&#34;: { &#34;total&#34;: { &#34;value&#34;: 1, &#34;relation&#34;: &#34;eq&#34; }, &#34;max_score&#34;: 1, &#34;hits&#34;: [ { &#34;_index&#34;: &#34;test-ignored&#34;, &#34;_id&#34;: &#34;qcf0wZABpEYH7Rw9OT7F&#34;, &#34;_score&#34;: 1, &#34;_ignored&#34;: [ &#34;length&#34; ], &#34;_source&#34;: { &#34;title&#34;: &#34;correct text&#34;, &#34;length&#34;: &#34;not a number&#34; } } ] } } 忽略指定字段 #  您可以使用 term 查询来查找特定字段被忽略的文档，如以下示例请求所示："
---


# _ignored 元数据字段

`_ignored` 字段帮助您管理文档中与格式错误数据相关的问题。由于在[索引映射](../mapping-parameters/_index.md)中启用了 `ignore_malformed` 设置，此字段用于存储在数据索引过程中被忽略的字段名称。

`_ignored` 字段允许您搜索和识别包含被忽略字段的文档，以及被忽略的具体字段名称。这对于故障排除很有用。

您可以使用 `term`、`terms` 和 `exists` 查询来查询 `_ignored` 字段。

> **注意**：只有当索引映射中启用了 `ignore_malformed` 设置时，才会填充 `_ignored` 字段。如果 `ignore_malformed` 设置为 `false`（默认值），则格式错误的字段将导致整个文档被拒绝，并且不会填写 `_ignored` 字段。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [元数据字段]({{< relref "/docs/features/mapping-and-analysis/metadata-field/_index.md" >}})

以下示例展示了如何使用 `_ignored` 字段：

```
GET _search
{
  "query": {
    "exists": {
      "field": "_ignored"
    }
  }
}
```

##### 使用 `_ignored` 字段的索引请求示例

以下示例向 `test-ignored` 索引添加一个新文档，其中 `ignore_malformed` 设置为 `true`，这样在数据索引时不会抛出错误：

```
PUT test-ignored
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "length": {
        "type": "long",
        "ignore_malformed": true
      }
    }
  }
}

POST test-ignored/_doc
{
  "title": "correct text",
  "length": "not a number"
}

GET test-ignored/_search
{
  "query": {
    "exists": {
      "field": "_ignored"
    }
  }
}
```

##### 示例返回内容

```
{
  "took": 42,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "test-ignored",
        "_id": "qcf0wZABpEYH7Rw9OT7F",
        "_score": 1,
        "_ignored": [
          "length"
        ],
        "_source": {
          "title": "correct text",
          "length": "not a number"
        }
      }
    ]
  }
}
```

### 忽略指定字段

您可以使用 `term` 查询来查找特定字段被忽略的文档，如以下示例请求所示：

```
GET _search
{
  "query": {
    "term": {
      "_ignored": "created_at"
    }
  }
}
```

##### 返回内容

```
{
  "took": 51,
  "timed_out": false,
  "_shards": {
    "total": 45,
    "successful": 45,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}


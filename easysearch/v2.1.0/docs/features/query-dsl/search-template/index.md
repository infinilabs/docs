---
title: "Search Template API"
date: 0001-01-01
summary: "Search Template API #  您可以将全文查询转换为查询模板，以接受用户输入并将其动态插入到查询中。
例如，如果您使用 Easysearch 作为应用程序或网站的后端搜索引擎，则可以从搜索栏或表单字段接收用户查询，并将其作为参数传递到查询模板中。这样，创建 Easysearch 查询的语法就从最终用户那里抽象出来了。
当您编写代码将用户输入转换为 Easysearch 查询时，可以使用查询模板简化代码。如果需要将字段添加到搜索查询中，只需修改模板即可，而无需更改代码。
查询模板使用 Mustache 语言。有关所有语法选项的列表，请参阅 Mustache 手册。
相关指南（先读这些） #    查询 DSL 基础  全文搜索  创建查询模版 #  查询模版有两个组件：查询和参数。参数是放置在变量中的用户输入值。在 Mustache 符号中，变量用双括号表示。当在查询中遇到类似 {% raw %}{{var}}{% endraw %} 的变量时，Easysearch 会转到 params 部分，查找名为 var 的参数，并用指定的值替换它。
您可以编写应用程序代码，询问用户要搜索什么，然后在运行时将该值插入 params 对象中。
此命令定义了一个查询模版，用于按名称查找播放。查询中的 {% raw %}{{play_name}}{% endraw %} 被值 Henry IV 替换：
GET _search/template { &#34;source&#34;: { &#34;query&#34;: { &#34;match&#34;: { &#34;play_name&#34;: &#34;{% raw %}{{play_name}}{% endraw %}&#34; } } }, &#34;params&#34;: { &#34;play_name&#34;: &#34;Henry IV&#34; } } 此模板在整个集群上运行搜索。"
---


# Search Template API

您可以将全文查询转换为查询模板，以接受用户输入并将其动态插入到查询中。

例如，如果您使用 Easysearch 作为应用程序或网站的后端搜索引擎，则可以从搜索栏或表单字段接收用户查询，并将其作为参数传递到查询模板中。这样，创建 Easysearch 查询的语法就从最终用户那里抽象出来了。

当您编写代码将用户输入转换为 Easysearch 查询时，可以使用查询模板简化代码。如果需要将字段添加到搜索查询中，只需修改模板即可，而无需更改代码。

查询模板使用 Mustache 语言。有关所有语法选项的列表，请参阅 [Mustache 手册](http://mustache.github.io/mustache.5.html)。

## 相关指南（先读这些）

- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

## 创建查询模版

查询模版有两个组件：查询和参数。参数是放置在变量中的用户输入值。在 Mustache 符号中，变量用双括号表示。当在查询中遇到类似 `{% raw %}{{var}}{% endraw %}` 的变量时，Easysearch 会转到 `params` 部分，查找名为 `var` 的参数，并用指定的值替换它。

您可以编写应用程序代码，询问用户要搜索什么，然后在运行时将该值插入 `params` 对象中。

此命令定义了一个查询模版，用于按名称查找播放。查询中的 `{% raw %}{{play_name}}{% endraw %}` 被值 `Henry IV` 替换：

```json
GET _search/template
{
  "source": {
    "query": {
      "match": {
        "play_name": "{% raw %}{{play_name}}{% endraw %}"
      }
    }
  },
  "params": {
    "play_name": "Henry IV"
  }
}
```

此模板在整个集群上运行搜索。

要在特定索引上运行此搜索，请将索引名称添加到请求中：

```json
GET shakespeare/_search/template
```

指定 `from` 和 `size` 参数：

```json
GET _search/template
{
  "source": {
    "from": "{% raw %}{{from}}{% endraw %}",
    "size": "{% raw %}{{size}}{% endraw %}",
    "query": {
      "match": {
        "play_name": "{% raw %}{{play_name}}{% endraw %}"
      }
    }
  },
  "params": {
    "play_name": "Henry IV",
    "from": 10,
    "size": 10
  }
}
```

为了改善搜索体验，您可以定义默认值，这样用户就不必指定每个可能的参数。如果参数未在 `params` 部分中定义，Easysearch 将使用默认值。

定义变量 `var` 默认值的语法如下：

```json
{% raw %}{{var}}{{^var}}default value{{/var}}{% endraw %}
```

此命令将 `from` 的默认值设置为 10，将 `size` 设置为 10：

```json
GET _search/template
{
  "source": {
    "from": "{% raw %}{{from}}{{^from}}10{{/from}}{% endraw %}",
    "size": "{% raw %}{{size}}{{^size}}10{{/size}}{% endraw %}",
    "query": {
      "match": {
        "play_name": "{% raw %}{{play_name}}{% endraw %}"
      }
    }
  },
  "params": {
    "play_name": "Henry IV"
  }
}
```

## 保存并执行查询模版

查询模版按您希望的方式工作后，可以将该模板的源保存为脚本，使其可用于不同的输入参数。

将查询模版另存为脚本时，需要将 `lang` 参数指定为 `muscle` ：

```json
POST _scripts/play_search_template
{
  "script": {
    "lang": "mustache",
    "source": {
      "from": "{% raw %}{{from}}{{^from}}0{{/from}}{% endraw %}",
      "size": "{% raw %}{{size}}{{^size}}10{{/size}}{% endraw %}",
      "query": {
        "match": {
          "play_name": "{{play_name}}"
        }
      }
    },
    "params": {
      "play_name": "Henry IV"
    }
  }
}
```

现在，您可以通过引用模板的 `id` 参数来重用模板。

您可以将此源模板用于不同的输入值。

```json
GET _search/template
{
  "id": "play_search_template",
  "params": {
    "play_name": "Henry IV",
    "from": 0,
    "size": 1
  }
}
```

#### 输出示例

```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 6,
    "successful": 6,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3205,
      "relation": "eq"
    },
    "max_score": 3.641852,
    "hits": [
      {
        "_index": "shakespeare",
        "_type": "_doc",
        "_id": "4",
        "_score": 3.641852,
        "_source": {
          "type": "line",
          "line_id": 5,
          "play_name": "Henry IV",
          "speech_number": 1,
          "line_number": "1.1.2",
          "speaker": "KING HENRY IV",
          "text_entry": "Find we a time for frighted peace to pant,"
        }
      }
    ]
  }
}
```

如果您有一个存储的模板并希望对其进行验证，请使用 `render` 操作：

```json
POST _render/template
{
  "id": "play_search_template",
  "params": {
    "play_name": "Henry IV"
  }
}
```

#### 输出示例

```json
{
  "template_output": {
    "from": "0",
    "size": "10",
    "query": {
      "match": {
        "play_name": "Henry IV"
      }
    }
  }
}
```

## 使用查询模版进行高级参数转换

Mustache 中有很多不同的语法选项，可以将输入参数转换为查询。

您可以指定条件、运行循环、连接数组、将数组转换为 JSON 等。

### 条件

使用 Mustache 中的条件表达方式：

```json
{% raw %}{{#var}}var{{/var}}{% endraw %}
```

当 `var` 是布尔值时，此语法充当 `if` 条件。只有当 `var` 的值为 `true` 时， `{#var}}` 和 `{/var}}}` 标记之间才会插入参数值。

使用 section 标记会使 JSON 无效，因此必须改用字符串格式编写查询。

只有当 `limit` 参数设置为 `true` 时，此命令才会在查询中包含 `size` 参数。

在以下示例中， `limit` 参数为 `true` ，因此 `size` 参数被激活。因此，您只会得到两个文档。

```json
GET _search/template
{
  "source": "{% raw %}{ {{#limit}} \"size\": \"{{size}}\", {{/limit}}  \"query\":{\"match\":{\"play_name\": \"{{play_name}}\"}}}{% endraw %}",
  "params": {
    "play_name": "Henry IV",
    "limit": true,
    "size": 2
  }
}
```

您还可以设计 `if-else` 条件。

如果 `limit` 为 `true` ，则此命令将 `size` 设置为 `2` 。否则，它将 `大小` 设置为 `10` 。

```json
GET _search/template
{
  "source": "{% raw %}{ {{#limit}} \"size\": \"2\", {{/limit}} {{^limit}} \"size\": \"10\", {{/limit}} \"query\":{\"match\":{\"play_name\": \"{{play_name}}\"}}}{% endraw %}",
  "params": {
    "play_name": "Henry IV",
    "limit": true
  }
}
```

### 循环

您还可以使用 section 标记来实现 foreach 循环：

```
{% raw %}{{#var}}{{.}}{{/var}}{% endraw %}
```

当 `var` 是一个数组时，查询模版遍历它并创建一个 `terms` 查询。

```json
GET _search/template
{
  "source": "{% raw %}{\"query\":{\"terms\":{\"play_name\":[\"{{#play_name}}\",\"{{.}}\",\"{{/play_name}}\"]}}}{% endraw %}",
  "params": {
    "play_name": [
      "Henry IV",
      "Othello"
    ]
  }
}
```

此模板呈现为：

```json
GET _search/template
{
  "source": {
    "query": {
      "terms": {
        "play_name": [
          "Henry IV",
          "Othello"
        ]
      }
    }
  }
}
```

### 关联

可以使用 `join` 标记连接数组的值（用逗号分隔）：

```json
GET _search/template
{
  "source": {
    "query": {
      "match": {
        "text_entry": "{% raw %}{{#join}}{{text_entry}}{{/join}}{% endraw %}"
      }
    }
  },
  "params": {
    "text_entry": [
      "To be",
      "or not to be"
    ]
  }
}
```

此模板呈现为:

```json
GET _search/template
{
  "source": {
    "query": {
      "match": {
        "text_entry": "{0=To be, 1=or not to be}"
      }
    }
  }
}
```

### 转换为 JSON

您可以使用 `toJson` 标记将参数转换为 JSON 表示形式：

```json
GET _search/template
{
  "source": "{\"query\":{\"bool\":{\"must\":[{\"terms\": {\"text_entries\": {% raw %}{{#toJson}}text_entries{{/toJson}}{% endraw %} }}] }}}",
  "params": {
    "text_entries": [
        { "term": { "text_entry" : "love" } },
        { "term": { "text_entry" : "soldier" } }
    ]
  }
}
```

渲染结果:

```json
GET _search/template
{
  "source": {
    "query": {
      "bool": {
        "must": [
          {
            "terms": {
              "text_entries": [
                {
                  "term": {
                    "text_entry": "love"
                  }
                },
                {
                  "term": {
                    "text_entry": "soldier"
                  }
                }
              ]
            }
          }
        ]
      }
    }
  }
}
```

## 多个查询模版

您可以捆绑多个查询模版，并使用 `msearch` 操作在单个请求中将它们发送到 Easysearch 集群。

这节省了网络往返时间，因此与独立请求相比，您可以更快地返回响应。

```json
GET _msearch/template
{"index":"shakespeare"}
{"id":"if_search_template","params":{"play_name":"Henry IV","limit":false,"size":2}}
{"index":"shakespeare"}
{"id":"play_search_template","params":{"play_name":"Henry IV"}}
```

## 管理查询模版

要列出所有脚本，请运行以下命令：

```json
GET _cluster/state/metadata?pretty&filter_path=**.stored_scripts
```

要检索特定查询模版，请运行以下命令：

```json
GET _scripts/<name_of_search_template>
```

要删除查询模版，请运行以下命令：

```json
DELETE _scripts/<name_of_search_template>
```


---
title: "搜索管道"
date: 0001-01-01
summary: "搜索管道 #  本节作为搜索管道相关 API 的 Reference 入口，聚焦于创建/更新管道及各类处理器字段说明。
如何从架构角度设计查询重写、结果增强与重排序策略，请优先阅读搜索与 AI 搜索相关 Guide。
相关指南（先读这些） #    搜索体验与高级功能  向量搜索  您可以使用搜索管道构建新的或重用现有的结果重排器、查询重写器以及其他对查询或结果进行操作的组件。搜索管道使您能够更轻松地在 Easysearch 中处理搜索查询和搜索结果。
将部分应用程序功能迁移到 Easysearch 搜索管道中可以降低应用程序的整体复杂性。作为搜索管道的一部分，您可以指定执行模块化任务的处理器列表，然后按需添加或重新排序这些处理器，以自定义应用程序的搜索结果。
名词解释 #  以下是与搜索管道相关的术语说明：
   名词名称 定义说明     查询重写处理器 用于拦截搜索请求（包含查询语句及请求中传递的元数据），在查询执行之前对原始查询进行重写和增强。   结果增强处理器 用于拦截搜索响应及搜索请求（包含查询语句、返回结果及请求中传递的元数据），对搜索响应执行相关操作后返回处理结果的组件。   重排序处理器 在协调节点层面运行于搜索阶段之间的组件。该处理器会对初步合并的搜索结果进行重新排序，提升相关性。   处理器（Processor） 泛指查询重写处理器或结果增强处理器。   搜索管道（Pipeline） 在 Easysearch 中集成的一个有序处理器列表。该管道会拦截查询，先对查询进行处理，再将其发往 Easysearch；随后拦截返回的结果，对结果进行处理，最后将结果返回给调用方。    可用处理器一览 #  查询重写处理器（Request Processors） #     处理器 类型标识 来源 说明      过滤查询处理器 filter_query 内置模块 向搜索请求添加额外的过滤条件    脚本处理器 script 内置模块 使用 Painless 脚本修改搜索请求参数    语义查询增强处理器 semantic_query_enricher AI 插件 为语义查询自动注入 Embedding 模型连接信息    结果增强处理器（Response Processors） #     处理器 类型标识 来源 说明      重命名字段处理器 rename_field 内置模块 对搜索结果中的字段进行重命名    混合评分解释处理器 hybrid_score_explanation AI 插件 为混合搜索结果添加归一化与融合的评分解释    重排序处理器（Search Phase Results Processors） #     处理器 类型标识 来源 说明      混合排序处理器 hybrid_ranker_processor AI 插件 使用 RRF 算法融合多路召回结果的排序     💡 AI 插件提供的处理器需要安装 AI 插件后方可使用。详细的 AI 搜索工作流请参阅 AI API 集成。"
---



# 搜索管道

本节作为搜索管道相关 API 的 Reference 入口，聚焦于创建/更新管道及各类处理器字段说明。  
如何从架构角度设计查询重写、结果增强与重排序策略，请优先阅读搜索与 AI 搜索相关 Guide。

## 相关指南（先读这些）

- [搜索体验与高级功能]({{< relref "/docs/features/query-dsl/_index.md" >}})
- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})

您可以使用搜索管道构建新的或重用现有的结果重排器、查询重写器以及其他对查询或结果进行操作的组件。搜索管道使您能够更轻松地在 Easysearch 中处理搜索查询和搜索结果。  
将部分应用程序功能迁移到 Easysearch 搜索管道中可以降低应用程序的整体复杂性。作为搜索管道的一部分，您可以指定执行模块化任务的处理器列表，然后按需添加或重新排序这些处理器，以自定义应用程序的搜索结果。

## 名词解释
以下是与搜索管道相关的术语说明：

| 名词名称           | 定义说明                                                                                         |
| -------------- | -------------------------------------------------------------------------------------------- |
| 查询重写处理器        | 用于拦截搜索请求（包含查询语句及请求中传递的元数据），在查询执行之前对原始查询进行重写和增强。                                              |
| 结果增强处理器        | 用于拦截搜索响应及搜索请求（包含查询语句、返回结果及请求中传递的元数据），对搜索响应执行相关操作后返回处理结果的组件。                                  |
| 重排序处理器         | 在协调节点层面运行于搜索阶段之间的组件。该处理器会对初步合并的搜索结果进行重新排序，提升相关性。                                             |
| 处理器（Processor） | 泛指查询重写处理器或结果增强处理器。                                                                           |
| 搜索管道（Pipeline） | 在 Easysearch 中集成的一个有序处理器列表。该管道会拦截查询，先对查询进行处理，再将其发往 Easysearch；随后拦截返回的结果，对结果进行处理，最后将结果返回给调用方。 |

## 可用处理器一览

### 查询重写处理器（Request Processors）

| 处理器 | 类型标识 | 来源 | 说明 |
|--------|---------|------|------|
| [过滤查询处理器](filter-query-processor.md) | `filter_query` | 内置模块 | 向搜索请求添加额外的过滤条件 |
| [脚本处理器](script-processor.md) | `script` | 内置模块 | 使用 Painless 脚本修改搜索请求参数 |
| [语义查询增强处理器](semantic-query-enricher-processor.md) | `semantic_query_enricher` | AI 插件 | 为语义查询自动注入 Embedding 模型连接信息 |

### 结果增强处理器（Response Processors）

| 处理器 | 类型标识 | 来源 | 说明 |
|--------|---------|------|------|
| [重命名字段处理器](rename-field-processor.md) | `rename_field` | 内置模块 | 对搜索结果中的字段进行重命名 |
| [混合评分解释处理器](hybrid-score-explanation-processor.md) | `hybrid_score_explanation` | AI 插件 | 为混合搜索结果添加归一化与融合的评分解释 |

### 重排序处理器（Search Phase Results Processors）

| 处理器 | 类型标识 | 来源 | 说明 |
|--------|---------|------|------|
| [混合排序处理器](hybrid-ranker-processor.md) | `hybrid_ranker_processor` | AI 插件 | 使用 RRF 算法融合多路召回结果的排序 |

> 💡 AI 插件提供的处理器需要安装 AI 插件后方可使用。详细的 AI 搜索工作流请参阅 [AI API 集成]({{< relref "/docs/integrations/ai/ai-api/" >}})。

{{% load-img "/img/search_pipelines.svg" %}}


## 创建搜索管道

搜索管道存储在集群状态中。要创建搜索管道，您必须在 Easysearch 集群中配置一个有序的处理器列表。管道中可以包含多个相同类型的处理器。每个处理器都有一个 `tag`（标签）标识符，用于将其与其他处理器区分开来。当调试错误信息时，为特定处理器设置标签非常有用，尤其是在添加了多个相同类型的处理器时。

#### API 说明

##### 请求语法
PUT /_search/pipeline/<pipeline-id>

##### 路径参数

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|--------|------|
| `pipeline-id` | 字符串 | 是 | 搜索管道的唯一标识符。命名规则：<br>• 仅允许小写字母、数字、连字符（`-`）、下划线（`_`）<br>• 不能以 `_` 开头<br>• 长度建议不超过 255 字符 |

##### 请求体参数

请求体是一个 JSON 对象，包含一个或多个处理器列表。所有字段均为**可选**，但至少需提供 `rewrite_processors` 或 `enrich_processors` 中的一个。

| 字段 | 类型 | 说明                                                  |
|------|------|-----------------------------------------------------|
| `rewrite_processors` | 数组 | **查询重写处理器列表**。在查询执行前对搜索请求进行修改（如添加过滤条件、重写查询结构等）。     |
| `enrich_processors` | 数组 | **结果增强处理器列表**。在搜索响应返回前对结果进行处理（如重命名字段、添加元数据等）。       |
| `rerank_processors` | 数组 | **重排序处理器列表**。在协调节点上对初步合并的搜索结果进行二次排序（如融合关键词与向量搜索结果）。 |

> 💡 **注意**：
> - 处理器按数组顺序依次执行。
> - 同一类型处理器可重复添加（建议使用 `tag` 区分）。
> - 若请求体为空 `{}`，将创建一个空管道（无实际作用）。


#### 示例请求

以下请求创建一个名为 `my_pipeline` 的搜索管道，其中包含一个 `filter_query` 查询重写处理器和一个结果增强处理器：

- 查询重写处理器使用 term 查询，仅返回可见性为“public”（公开）的消息；
- 结果增强处理器将字段 `message` 重命名为 `notification`。

```auto
PUT /_search/pipeline/my_pipeline
{
  "rewrite_processors": [
    {
      "filter_query": {
        "tag": "tag1",
        "description": "此处理器将限制只返回公开可见的文档",
        "query": {
          "term": {
            "visibility": "public"
          }
        }
      }
    }
  ],
  "enrich_processors": [
    {
      "rename_field": {
        "field": "message",
        "target_field": "notification"
      }
    }
  ]
}
```

### 忽略处理器失败

默认情况下，如果搜索管道中的某个处理器执行失败，整个管道将停止运行。如果您希望在某个处理器失败时管道仍继续执行后续处理器，可以在创建管道时将该处理器的 `ignore_failure` 参数设置为 `true`：

```auto
"filter_query" : {
  "tag" : "tag1",
  "description" : "This processor is going to restrict to publicly visible documents",
  "ignore_failure": true,
  "query" : {
    "term": {
      "visibility": "public"
    }
  }
}
```

当设置了 `ignore_failure: true` 后，如果该处理器执行失败，Easysearch 会记录失败日志，并继续执行搜索管道中剩余的其他处理器。

## 更新搜索管道

要动态更新搜索管道，可以通过 Search Pipeline API 重新替换该管道的配置。

#### 示例请求

以下示例请求通过添加一个 `filter_query` 查询重写处理器和一个 `rename_field` 结果增强处理器，对 `my_pipeline` 进行 upsert（存在则更新，不存在则创建）操作：

```auto
PUT /_search/pipeline/my_pipeline
{
  "rewrite_processors": [
    {
      "filter_query": {
        "tag": "tag1",
        "description": "此处理器仅返回公开可见的文档",
        "query": {
          "term": {
            "visibility": "public"
          }
        }
      }
    }
  ],
  "enrich_processors": [
    {
      "rename_field": {
        "field": "message",
        "target_field": "notification"
      }
    }
  ]
}
```

> ⚠️ 注意：更新操作会完全覆盖原有管道配置，因此请确保在请求体中包含更新后所需的全部处理器。


## 使用搜索管道

有2种方式使用搜索管道：
- [为请求指定一个已存在的搜索管道](#为请求指定一个已存在的搜索管道)。
- 为索引中的所有请求设置一个[默认搜索管道](#默认搜索管道)。

### 为请求指定一个已存在的搜索管道

在你[创建搜索管道](#创建搜索管道)之后，可以通过在查询中指定 `search_pipeline` 查询参数来使用该管道：

```auto
GET /my_index/_search?search_pipeline=my_pipeline
```

## 默认搜索管道

为了方便起见，你可以为某个索引设置一个默认的搜索管道。一旦设置了默认管道，就无需在每次搜索请求中显式指定 `search_pipeline` 参数。

### 为索引设置默认搜索管道

要为索引设置默认搜索管道，可以在索引设置中指定 `index.search.default_pipeline`：

```auto
PUT /my_index/_settings
{
  "index.search.default_pipeline": "my_pipeline"
}
```

为 `my_index` 设置默认管道后，你可以执行一个简单的全文搜索请求：

```auto
GET /my_index/_search
```

响应中仅包含公开文档，说明默认管道已自动应用：

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "took": 19,
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
    "max_score": 0.0,
    "hits": [
      {
        "_index": "my_index",
        "_id": "1",
        "_score": 0.0,
        "_source": {
          "message": "这是一个公开消息",
          "visibility": "public"
        }
      }
    ]
  }
}
```
</details>

### 在请求中禁用默认管道

如果你希望执行一个不应用默认管道的搜索请求，可以将 `search_pipeline` 查询参数设置为 `_none`：

```auto
GET /my_index/_search?search_pipeline=_none
```

### 移除默认管道

要从索引中移除默认管道，可将其设置为 `null` 或 `_none`：

```auto
PUT /my_index/_settings
{
  "index.search.default_pipeline": null
}
```

```auto
PUT /my_index/_settings
{
  "index.search.default_pipeline": "_none"
}
```

## 删除搜索管道
要删除特定搜索管道，请将管道 ID 作为参数传递：

```auto
DELETE /_search/pipeline/<pipeline-id>
```

要删除集群中的所有搜索管道，请使用通配符 ( * )：
```auto
DELETE /_search/pipeline/*
```

## 检索搜索管道

要查看所有搜索管道，可发送以下请求：

```auto
GET /_search/pipeline
```

响应内容包含你在上一节中创建的管道：

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "my_pipeline": {
    "rewrite_processors": [
      {
        "filter_query": {
          "tag": "tag1",
          "description": "该处理器将限制只返回公开可见的文档",
          "query": {
            "term": {
              "visibility": "public"
            }
          }
        }
      }
    ]
  }
}
```
</details>

要查看某个特定的管道，可在路径中指定管道名称作为参数：

```auto
GET /_search/pipeline/my_pipeline
```

你也可以使用通配符模式来查看部分管道，例如：

```auto
GET /_search/pipeline/my*
```

这将返回所有以 `my` 开头的搜索管道。

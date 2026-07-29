---
title: "脚本指标聚合（Scripted Metric）"
date: 0001-01-01
summary: "脚本指标聚合 #  scripted_metric 聚合是一个多值指标聚合，它返回根据指定脚本计算的指标。脚本有四个阶段：init、map、combine 和 reduce，每个聚合按顺序运行这些阶段，组合来自文档的结果。
相关指南（先读这些） #    聚合基础  聚合场景实践  所有四个脚本共享一个可变对象，称为 state，该对象由你定义。state 在 init、map 和 combine 阶段时对每个分片是局部的。结果被传递到 states 数组中用于 reduce 阶段。因此，每个分片的 state 在分片在 reduce 步骤中组合之前是独立的。
参数说明 #  scripted_metric 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     init_script 可选 String 一个脚本，在每个分片上处理任何文档之前执行一次。用于设置初始 state（例如，在 state 对象中初始化计数器或列表）。如果没有提供，state 在每个分片上开始时是一个空对象。   map_script 必需 String 一个脚本，针对聚合收集的每个文档执行。此脚本根据文档的数据更新 state。例如，您可以检查字段的值，然后递增计数器或在 state 中计算运行总和。   combine_script 必需 String 一个脚本，在每个分片上处理完所有文档后由 map_script 执行一次。此脚本将分片的 state 聚合为单个结果发送回协调节点。此脚本用于完成一个分片的计算（例如，汇总存储在 state 中的计数器或总计）。脚本应返回其分片的汇总值或结构。   reduce_script 必需 String 一个脚本在接收到所有分片的合并结果后，在协调节点上执行一次。这个脚本接收一个特殊变量 states，它是一个包含每个分片从 combine_script 输出的数组。reduce_script 遍历状态并生成最终的聚合输出（例如，添加分片总和或合并计数的映射）。reduce_script 返回的值是在聚合结果中报告的值。   params 可选 Object 除 reduce_script 外，所有脚本都可以访问用户定义的参数。    可返回的类型 #  脚本可以在内部使用任何有效的操作和对象。然而，存储在 state 或从任何脚本返回的数据必须属于允许的类型之一。这个限制存在是因为中间 state 需要在节点之间发送。允许的类型如下："
---


# 脚本指标聚合

`scripted_metric` 聚合是一个多值指标聚合，它返回根据指定脚本计算的指标。脚本有四个阶段：`init`、`map`、`combine` 和 `reduce`，每个聚合按顺序运行这些阶段，组合来自文档的结果。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

所有四个脚本共享一个可变对象，称为 `state`，该对象由你定义。`state` 在 `init`、`map` 和 `combine` 阶段时对每个分片是局部的。结果被传递到 `states` 数组中用于 `reduce` 阶段。因此，每个分片的 `state` 在分片在 `reduce` 步骤中组合之前是独立的。

## 参数说明

scripted_metric 聚合采用以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| --------- | --------- | -------- | ---------------------------------------------------------------------- |
| `init_script` | 可选 | String | 一个脚本，在每个分片上处理任何文档之前执行一次。用于设置初始 `state`（例如，在 `state` 对象中初始化计数器或列表）。如果没有提供，`state` 在每个分片上开始时是一个空对象。|
| `map_script` | 必需 | String | 一个脚本，针对聚合收集的每个文档执行。此脚本根据文档的数据更新 `state`。例如，您可以检查字段的值，然后递增计数器或在 `state` 中计算运行总和。|
| `combine_script` | 必需 | String | 一个脚本，在每个分片上处理完所有文档后由 `map_script` 执行一次。此脚本将分片的 `state` 聚合为单个结果发送回协调节点。此脚本用于完成一个分片的计算（例如，汇总存储在 `state` 中的计数器或总计）。脚本应返回其分片的汇总值或结构。|
| `reduce_script` | 必需 | String | 一个脚本在接收到所有分片的合并结果后，在协调节点上执行一次。这个脚本接收一个特殊变量 `states`，它是一个包含每个分片从 `combine_script` 输出的数组。`reduce_script` 遍历状态并生成最终的聚合输出（例如，添加分片总和或合并计数的映射）。`reduce_script` 返回的值是在聚合结果中报告的值。|
| `params` | 可选 | Object | 除 `reduce_script` 外，所有脚本都可以访问用户定义的参数。|

## 可返回的类型

脚本可以在内部使用任何有效的操作和对象。然而，存储在 `state` 或从任何脚本返回的数据必须属于允许的类型之一。这个限制存在是因为中间 `state` 需要在节点之间发送。允许的类型如下：

- 基本类型： `int` , `long` , `float` , `double` , `boolean`
- `String` 字符串
- `Map` 映射（键和值仅允许为允许类型：基本类型、字符串、映射或数组）
- 数组（仅包含允许类型：基本类型、字符串、映射或数组）

`state` 可以是一个数字、字符串、`map`（对象）或数组（列表），也可以是这些的组合。例如，你可以使用 `map` 来累积多个计数器，使用数组来收集值，或使用单个数字来保持累计总和。如果你需要返回多个指标，可以将它们存储在 `map` 或数组中。如果你从 `reduce_script` 返回 `map` 作为最终值，聚合结果包含一个对象。如果你返回单个数字或字符串，结果是一个单一值。

## 在脚本中使用参数

您可以使用 `params` 字段可选地向脚本传递自定义参数。这是一个用户定义的对象，其内容成为 `init_script`、`map_script` 和 `combine_script` 中的变量。`reduce_script` 不会直接接收 `params`，因为在 `reduce` 阶段，所有需要的数据都必须在 `states` 数组中。如果您需要在 `reduce` 阶段使用常量，可以将其作为每个分片 `state` 的一部分，或使用存储的脚本。所有参数都必须在全局 `params` 对象中定义。这确保它们在不同脚本阶段之间共享。如果您未指定任何 `params`，则 `params` 对象为空。

例如，您可以在 `params` 中提供 `threshold` 或 `field` 名称，然后在脚本中引用 `params.threshold` 或 `params.field` ：

```
"scripted_metric": {
  "params": {
    "threshold": 100,
    "field": "amount"
  },
  "init_script": "...",
  "map_script": "...",
  "combine_script": "...",
  "reduce_script": "..."
}

```

## 参考样例

以下示例展示了使用 scripted_metric 的不同方法。

### 计算交易净利润

以下示例展示了如何使用 `scripted_metric` 聚合来计算内置聚合不直接支持的定制指标。该数据集表示财务交易，其中每个文档被分类为 `sale` （收入）或 `cost` （支出），并包含一个 `amount` 字段。目标是通过从所有文档的总销售额中减去总成本来计算总净利润。

创建索引：

```
PUT transactions
{
  "mappings": {
    "properties": {
      "type":   { "type": "keyword" },
      "amount": { "type": "double" }
    }
  }
}
```

索引四笔交易，两笔销售（金额 80 和 130 ），以及两笔成本（ 10 和 30 ）：

```
PUT transactions/_bulk?refresh=true
{ "index": {} }
{ "type": "sale", "amount": 80 }
{ "index": {} }
{ "type": "cost", "amount": 10 }
{ "index": {} }
{ "type": "cost", "amount": 30 }
{ "index": {} }
{ "type": "sale", "amount": 130 }
```

要运行一个使用 `scripted_metric` 聚合来计算利润的搜索，请使用以下脚本：

- `init_script` 创建一个空列表，用于存储每个分片的事务值。
- `map_script` 根据类型 `sale` 将每个文档的金额作为正数添加到 `state.transactions` 列表中，如果类型是 `cost` 则作为负数。在 `map` 阶段结束时，每个分片都有一个 `state.transactions` 列表，代表其收入和支出。
- `combine_script` 处理 `state.transactions` 列表，并为分片计算一个 `shardProfit` 值。然后返回 `shardProfit` 作为分片的输出。
- `reduce_script` 在协调节点上运行，接收 `states` 数组，其中包含每个分片的 `shardProfit` 值。它检查空条目，将这些值相加以计算总利润，并返回最终结果。

以下请求包含上面描述的脚本：

```
GET transactions/_search
{
  "size": 0,
  "aggs": {
    "total_profit": {
      "scripted_metric": {
        "init_script": "state.transactions = []",
        "map_script": "state.transactions.add(doc['type'].value == 'sale' ? doc['amount'].value : -1 * doc['amount'].value)",
        "combine_script": "double shardProfit = 0; for (t in state.transactions) { shardProfit += t; } return shardProfit;",
        "reduce_script": "double totalProfit = 0; for (p in states) { if (p != null) { totalProfit += p; }} return totalProfit;"
      }
    }
  }
}
```

返回 `total_profit` :

```
{
  ...
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "total_profit": {
      "value": 170
    }
  }
}

```

### 给 HTTP 响应代码分类

以下示例展示了如何使用 `scripted_metric` 聚合在单个聚合中返回多个值。数据集由包含 HTTP 响应码的 Web 服务器日志条目组成。目标是将这些响应分为三类：成功响应（2xx 状态码）、客户端或服务器错误（4xx 或 5xx 状态码）以及其他响应（1xx 或 3xx 状态码）。这种分类是通过在基于映射的聚合 `state` 中维护计数器来实现的。

创建一个示例索引：

```
PUT logs
{
  "mappings": {
    "properties": {
      "response": { "type": "keyword" }
    }
  }
}
```

添加具有各种响应代码的示例文档：

```
PUT logs/_bulk?refresh=true
{ "index": {} }
{ "response": "200" }
{ "index": {} }
{ "response": "201" }
{ "index": {} }
{ "response": "404" }
{ "index": {} }
{ "response": "500" }
{ "index": {} }
{ "response": "304" }

```

`state` （每个分片上）是一个 `map` ，包含三个计数器： `error` 、 `success` 和 `other` 。

要运行一个用于统计类别的脚本指标聚合，请使用以下脚本：

- `init_script` 将 `error` 、 `success` 和 `other` 的计数器初始化为 0 。
- `map_script` 检查每个文档的响应代码，并根据响应代码递增相应的计数器。
- `combine_script` 返回该分片的 `state.responses map` 。
- `reduce_script` 合并所有分片的 `maps` 数组 ( `states` )。因此，它创建一个新的组合 `map` ，并将每个分片的 `map` 中的 `error` 、 `success` 和 `other` 计数添加进去。这个组合的 `map` 作为最终结果返回。

以下请求包含了上面描述的脚本：

```
GET logs/_search
{
  "size": 0,
  "aggs": {
    "responses_by_type": {
      "scripted_metric": {
        "init_script": "state.responses = new HashMap(); state.responses.put('success', 0); state.responses.put('error', 0); state.responses.put('other', 0);",
        "map_script": """
          String code = doc['response'].value;
          if (code.startsWith("5") || code.startsWith("4")) {
            // 4xx or 5xx -> count as error
            state.responses.error += 1;
          } else if (code.startsWith("2")) {
            // 2xx -> count as success
            state.responses.success += 1;
          } else {
            // anything else (e.g., 1xx, 3xx, etc.) -> count as other
            state.responses.other += 1;
          }
        """,
        "combine_script": "return state.responses;",
        "reduce_script": """
          Map combined = new HashMap();
          combined.error = 0;
          combined.success = 0;
          combined.other = 0;
          for (state in states) {
            if (state != null) {
              combined.error += state.error;
              combined.success += state.success;
              combined.other += state.other;
            }
          }
          return combined;
        """
      }
    }
  }
}

```

在 `value` 对象中返回了三个值，展示了通过在 `state` 中使用 `map` ，脚本指标如何一次性返回多个指标

```
{
  ...
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "responses_by_type": {
      "value": {
        "other": 1,
        "success": 2,
        "error": 2
      }
    }
  }
}
```

## 管理空分组（没有文档）

当使用 `scripted_metric` 聚合作为分组聚合（例如 `terms` ）的子聚合时，需要考虑某些分片上不包含文档的分组。在这种情况下，这些分片会为聚合 `state` 返回 `null` 值。在 `reduce_script` 阶段， `states` 数组可能因此包含对应这些分片的 `null` 条目。为确保可靠执行， `reduce_script` 必须设计为能够优雅地处理 `null` 值。常见的方法是在访问或操作每个 `state` 之前加入条件检查，例如 `if (state != null)` 。若未实施此类检查，在跨分片处理空分组时可能导致运行时错误。

## 性能考量

由于脚本指标为每个文档运行自定义代码，因此可能会在内存中积累大量的 `state` ，所以它们可能比内置聚合慢。每个分片的中间 `state` 必须序列化才能发送到协调节点。因此，如果您的 `state` 非常大，它可能会消耗大量内存和网络带宽。为了保持搜索效率，请尽可能使您的脚本轻量，并避免在 `state` 中积累不必要的数据。在发送之前，使用合并阶段来缩减 `state` 数据，如“从交易中计算净利润”所示，并且仅收集您真正需要以生成最终指标的数据。


---
title: "Rewrite 参数"
date: 0001-01-01
summary: "Rewrite 参数 #  像 wildcard、prefix、regexp、fuzzy 和 range 这样的多词查询在内部会重组成一组词。rewrite 参数允许你控制这些词重写的执行和评分。
当多词查询扩展成很多词（例如 prefix: &quot;error*&quot; 匹配数百个词）时，它们在内部会转换成 term 查询。这个过程可能会有以下缺点：
 超出 indices.query.bool.max_clause_count 限制（默认是 1024）。 影响匹配文档的评分计算方式。 根据所使用的重写方法，影响内存和延迟。  rewrite 参数让你能够控制多词查询的内部行为。
   模式 评分规则 性能 注释     constant_score 所有匹配具有相同分数 最佳 默认模式，适合过滤器   scoring_boolean 基于 TF/IDF 中等 完整相关性评分   constant_score_boolean 相同分数但使用布尔结构 中等 与 must_not 或 minimum_should_match 一起使用   top_terms_N 在顶部 N 个词上使用 TF/IDF 高效 截断扩展   top_terms_boost_N 静态提升 快速 较低准确度   top_terms_blended_freqs_N 混合评分 平衡 最佳评分/效率权衡    相关指南（先读这些） #    部分匹配  查询 DSL 基础  可用的重写方法 #  下表总结了可用的重写方法。"
---


# Rewrite 参数

像 `wildcard`、`prefix`、`regexp`、`fuzzy` 和 `range` 这样的多词查询在内部会重组成一组词。`rewrite` 参数允许你控制这些词重写的执行和评分。

当多词查询扩展成很多词（例如 `prefix: "error*"` 匹配数百个词）时，它们在内部会转换成 `term` 查询。这个过程可能会有以下缺点：

- 超出 `indices.query.bool.max_clause_count` 限制（默认是 1024）。
- 影响匹配文档的评分计算方式。
- 根据所使用的重写方法，影响内存和延迟。

`rewrite` 参数让你能够控制多词查询的内部行为。

| 模式                        | 评分规则                   | 性能 | 注释                                         |
| --------------------------- | -------------------------- | ---- | -------------------------------------------- |
| `constant_score`            | 所有匹配具有相同分数       | 最佳 | 默认模式，适合过滤器                         |
| `scoring_boolean`           | 基于 TF/IDF                | 中等 | 完整相关性评分                               |
| `constant_score_boolean`    | 相同分数但使用布尔结构     | 中等 | 与 `must_not` 或 `minimum_should_match` 一起使用 |
| `top_terms_N`               | 在顶部 N 个词上使用 TF/IDF | 高效 | 截断扩展                                     |
| `top_terms_boost_N`         | 静态提升                   | 快速 | 较低准确度                                   |
| `top_terms_blended_freqs_N` | 混合评分                   | 平衡 | 最佳评分/效率权衡                            |

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 可用的重写方法

下表总结了可用的重写方法。
|重写方法 |描述|
|--------|----|
|`constant_score`|（默认）所有展开的词项作为一个单一单元一起被评估，为每个匹配分配相同的分数。匹配的文档不会被单独评分，这使得它在过滤用例中非常高效。|
|`scoring_boolean`|将查询分解为一个布尔 `should` 子句，每个匹配都有一个词项查询。每个结果根据相关性单独评分。|
|`constant_score_boolean`|类似于 `scoring_boolean` ，但所有文档都获得一个固定的分数，而不管词项频率如何。保持布尔结构，不进行 TF/IDF 加权。|
|`top_terms_N`|限制评分和执行到 N 个最频繁的词项。减少资源使用并防止子句过载。|
|`top_terms_boost_N`|与 top_terms_N 类似，但使用静态提升代替完整评分。通过简化相关性提供性能改进。|
|`top_terms_blended_freqs_N`|选择最匹配的前 N 个词项，并对其文档频率进行平均以用于评分。产生平衡的评分，无需完整词项爆炸。|

## 基于布尔值的重写限制

所有基于布尔值的重写，如 `scoring_boolean` 、 `constant_score_boolean` 和 `top_terms_*` ，都受以下动态集群级别的索引设置约束：

```
indices.query.bool.max_clause_count

```

此设置控制允许的最大布尔子句数量（默认： 1024 ）。如果您的查询扩展到的词项数量超过此限制，则会以 `too_many_clauses` 错误被拒绝。

例如，通配符（如“error\*”）可能会扩展到数百或数千个匹配词项，其中包括“error”、“errors”、“error_log”、“error404”等。每个词项都会变成一个单独的 `term` 查询。如果词项数量超过 `indices.query.bool.max_clause_count` 限制，查询将失败：

```
POST /logs/_search
{
  "query": {
    "wildcard": {
      "message": {
        "value": "error*",
        "rewrite": "scoring_boolean"
      }
    }
  }
}

```

查询的内部扩展如下：

```
{
  "bool": {
    "should": [
      { "term": { "message": "error" } },
      { "term": { "message": "errors" } },
      { "term": { "message": "error_log" } },
      { "term": { "message": "error404" } },
      ...
    ]
  }
}
```

## 常量评分

默认的 `constant_score` 重写方法将所有展开的词项包装成一个查询，并完全跳过评分阶段。这种方法具有以下特点：

- 将所有词项匹配作为单个位图查询执行。
- 完全忽略评分；每个文档都得到 `_score` 的 1.0 。
- 最快的选项；非常适合过滤。

以下示例使用默认的 `constant_score` 重写方法运行 `wildcard` 查询，以高效地过滤在 `message` 字段中匹配 `warning*` 模式的文档：

```
POST /logs/_search
{
  "query": {
    "wildcard": {
      "message": {
        "value": "warning*"
      }
    }
  }
}
```

## 评分布尔值

`scoring_boolean` 重写方法将扩展的词项拆分为单独的 `term` 查询，并在布尔 `should` 子句下组合。这种方法的工作原理如下：

- 将通配符扩展为布尔 `should` 子句内的单个 `term` 查询。
- 每个文档的评分反映了它与多少词项匹配以及这些词项的频率。

以下示例使用 `scoring_boolean` 重写配置：

```
POST /logs/_search
{
  "query": {
    "wildcard": {
      "message": {
        "value": "warning*",
        "rewrite": "scoring_boolean"
      }
    }
  }
}
```

## 常量分数布尔值

`constant_score_boolean` 重写方法使用与 `scoring_boolean` 相同的布尔结构，但禁用评分，因此当需要条件逻辑而不需要相关性排序时很有用。此方法具有以下特点：

- 与 `scoring_boolean` 结构相似，但文档不会被排序。
- 所有匹配的文档获得相同的分数。
- 保留布尔条件逻辑的灵活性，例如使用 `must_not` ，而不进行排序。

以下示例查询使用了一个 `must_not` 布尔子句：

```
POST /logs/_search
{
  "query": {
    "bool": {
      "must_not": {
        "wildcard": {
          "message": {
            "value": "error*",
            "rewrite": "constant_score_boolean"
          }
        }
      }
    }
  }
}

```

该查询内部展开如下：

```
{
  "bool": {
    "must_not": {
      "bool": {
        "should": [
          { "term": { "message": "error" } },
          { "term": { "message": "errors" } },
          { "term": { "message": "error_log" } },
          ...
        ]
      }
    }
  }
}
```

## 最佳词项 N

`top_terms_N` 方法是多种重写选项中的一种，旨在扩展多词查询时平衡评分准确性和性能。其工作原理如下：

- 仅选择并评分 N 个最频繁匹配的词。
- 当你预期会有大量词扩展且希望限制负载时，此方法很有用。
- 其他有效词会被忽略以保持性能。

以下查询使用 `top_terms_2` 重写方法，仅对在 `message` 字段中匹配 `warning*` 模式的两个最频繁的词进行评分：

```
POST /logs/_search
{
  "query": {
    "wildcard": {
      "message": {
        "value": "warning*",
        "rewrite": "top_terms_2"
      }
    }
  }
}

```

## 最佳词项评分 N

`top_terms_boost_N` 重写方法选择最匹配的前 N 个词项，并使用静态 `boost` 值代替计算完整的相关性分数。它的工作原理如下：

- 将扩展限制为最前 N 个词项，如 `top_terms_N` 。
- 它不是计算 TF/IDF，而是为每个词项分配预设的提升值。
- 提供更快的执行速度和可预测的相关性权重。

以下示例使用 `top_terms_boost_2` 重写参数：

```
POST /logs/_search
{
  "query": {
    "wildcard": {
      "message": {
        "value": "warning*",
        "rewrite": "top_terms_boost_2"
      }
    }
  }
}

```

## 混合频率的最佳词项 N

`top_terms_blended_freqs_N` 重写方法选择顶部 N 个匹配词项，并将它们的文档频率混合起来，以生成更平衡的相关性分数。这种方法具有以下特点：

- 选择顶部 N 个匹配词项，并对所有词项应用混合频率。
- 混合使不同频率的词项之间的评分更加平滑。
- 在追求性能与真实评分之间取得良好平衡时。

以下示例使用 `top_terms_blended_freqs_2` 重写参数：

```
POST /logs/_search
{
  "query": {
    "wildcard": {
      "message": {
        "value": "warning*",
        "rewrite": "top_terms_blended_freqs_2"
      }
    }
  }
}

```


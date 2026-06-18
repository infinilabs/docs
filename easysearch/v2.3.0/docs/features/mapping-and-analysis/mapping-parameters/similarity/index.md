---
title: "相似度算法参数（Similarity）"
date: 0001-01-01
summary: "Similarity 参数 #  similarity 参数指定字段使用的相关性评分算法。不同的算法适合不同类型的数据和搜索场景。
相关指南（先读这些） #    评分基础  映射基础  内置算法 #     值 算法 说明     BM25 Okapi BM25 默认值。适合大多数全文搜索场景。   boolean 布尔模型 不计算相关性分数，匹配的文档得分为查询的 boost 值。适合不需要相关性排序的过滤场景。   DFR Divergence from Randomness 基于随机性散度模型的评分算法。   DFI Divergence from Independence 基于独立性散度模型的评分算法。   IB Information Based 基于信息论的评分算法。   LMDirichlet Dirichlet 语言模型 使用 Dirichlet 先验的语言模型平滑方法。   LMJelinekMercer Jelinek-Mercer 语言模型 使用线性插值的语言模型平滑方法。   自定义名称 自定义相似度 在索引 settings 中定义的自定义评分算法。    示例 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;similarity&#34;: &#34;BM25&#34; }, &#34;status&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;similarity&#34;: &#34;boolean&#34; } } } } BM25 参数调优 #  BM25 的行为可以通过索引 settings 自定义："
---


# Similarity 参数

`similarity` 参数指定字段使用的相关性评分算法。不同的算法适合不同类型的数据和搜索场景。

## 相关指南（先读这些）

- [评分基础]({{< relref "/docs/features/fulltext-search/relevance/scoring-basics.md" >}})
- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})

## 内置算法

| 值 | 算法 | 说明 |
|----|------|------|
| `BM25` | Okapi BM25 | **默认值**。适合大多数全文搜索场景。 |
| `boolean` | 布尔模型 | 不计算相关性分数，匹配的文档得分为查询的 boost 值。适合不需要相关性排序的过滤场景。 |
| `DFR` | Divergence from Randomness | 基于随机性散度模型的评分算法。 |
| `DFI` | Divergence from Independence | 基于独立性散度模型的评分算法。 |
| `IB` | Information Based | 基于信息论的评分算法。 |
| `LMDirichlet` | Dirichlet 语言模型 | 使用 Dirichlet 先验的语言模型平滑方法。 |
| `LMJelinekMercer` | Jelinek-Mercer 语言模型 | 使用线性插值的语言模型平滑方法。 |
| `自定义名称` | 自定义相似度 | 在索引 settings 中定义的自定义评分算法。 |

## 示例

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": "BM25"
      },
      "status": {
        "type": "text",
        "similarity": "boolean"
      }
    }
  }
}
```

## BM25 参数调优

BM25 的行为可以通过索引 settings 自定义：

```json
PUT my-index
{
  "settings": {
    "similarity": {
      "custom_bm25": {
        "type": "BM25",
        "k1": 1.2,
        "b": 0.75
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": "custom_bm25"
      }
    }
  }
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `k1` | 1.2 | 词频饱和度。值越大，词频的影响越大。 |
| `b` | 0.75 | 字段长度归一化。0 表示不考虑长度，1 表示完全归一化。 |

## 何时更改 similarity

| 场景 | 建议 |
|------|------|
| 标准全文搜索 | 使用默认 `BM25` |
| 纯过滤/精确匹配场段 | 使用 `boolean`，减少评分计算开销 |
| 短文本字段（如标题） | 调低 `b` 值，减少长度影响 |
| 长文本字段（如正文） | 保持默认 `b` 值 |
---

## 其他内置相似度算法

### DFR（Divergence from Randomness）

基于"词频偏离随机分布的程度"来计算相关性。模型由三部分组成：基本模型、后置信息量模型和归一化方法。

```json
PUT my-index
{
  "settings": {
    "similarity": {
      "my_dfr": {
        "type": "DFR",
        "basic_model": "g",
        "after_effect": "l",
        "normalization": "h2",
        "normalization.h2.c": "3.0"
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": "my_dfr"
      }
    }
  }
}
```

| 参数 | 可选值 | 说明 |
|------|--------|------|
| `basic_model` | `be`, `d`, `g`, `if`, `in`, `ine`, `p` | 基本模型 |
| `after_effect` | `no`, `b`, `l` | 后置信息量模型 |
| `normalization` | `no`, `h1`, `h2`, `h3`, `z` | 归一化方法 |

归一化参数（按所选归一化方法）：

| 参数 | 说明 |
|------|------|
| `normalization.h1.c` | `h1` 归一化的 c 参数（默认 1） |
| `normalization.h2.c` | `h2` 归一化的 c 参数（默认 1） |
| `normalization.h3.c` | `h3` 归一化的 c 参数（默认 800） |
| `normalization.z.z` | `z` 归一化的 z 参数（默认 0.3） |

### DFI（Divergence from Independence）

基于词频偏离"独立假设"的程度来计算相关性。对于短文本和精确匹配场景效果较好。

```json
PUT my-index
{
  "settings": {
    "similarity": {
      "my_dfi": {
        "type": "DFI",
        "independence_measure": "standardized"
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": "my_dfi"
      }
    }
  }
}
```

| 参数 | 可选值 | 说明 |
|------|--------|------|
| `independence_measure` | `standardized`, `saturated`, `chisquared` | 独立性度量方法 |

### IB（Information Based）

基于信息论原理，将词频看作信息内容的度量。由分布模型和 Lambda 参数构成。

```json
PUT my-index
{
  "settings": {
    "similarity": {
      "my_ib": {
        "type": "IB",
        "distribution": "ll",
        "lambda": "df",
        "normalization": "h2",
        "normalization.h2.c": "3.0"
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": "my_ib"
      }
    }
  }
}
```

| 参数 | 可选值 | 说明 |
|------|--------|------|
| `distribution` | `ll`, `spl` | 分布模型（log-logistic 或 smoothed power-law） |
| `lambda` | `df`, `ttf` | Lambda 参数（文档频率或总词频） |
| `normalization` | 同 DFR | 归一化方法 |

### LMDirichlet（Dirichlet 语言模型）

使用 Dirichlet 先验进行平滑的语言模型方法。对短文本和小文档集表现较好。

```json
PUT my-index
{
  "settings": {
    "similarity": {
      "my_lm_dirichlet": {
        "type": "LMDirichlet",
        "mu": 2000
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": "my_lm_dirichlet"
      }
    }
  }
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `mu` | 2000 | Dirichlet 先验参数。值越大，平滑效果越强。 |

### LMJelinekMercer（Jelinek-Mercer 语言模型）

使用线性插值将文档模型与集合模型混合的语言模型方法。适合长查询文本。

```json
PUT my-index
{
  "settings": {
    "similarity": {
      "my_lm_jm": {
        "type": "LMJelinekMercer",
        "lambda": 0.1
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": "my_lm_jm"
      }
    }
  }
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `lambda` | 0.1 | 插值权重。0 = 仅文档模型，1 = 仅集合模型。较小的值偏重文档，较大的值偏重集合频率。 |

---

## 算法选择建议

| 算法 | 适用场景 | 特点 |
|------|----------|------|
| **BM25** | 通用全文搜索 | 默认选择，鲁棒性最强 |
| **boolean** | 过滤/精确匹配 | 无评分开销 |
| **DFR** | 学术/高级调优 | 多参数组合，灵活性强 |
| **DFI** | 短文本搜索 | 基于独立性假设 |
| **IB** | 信息检索研究 | 基于信息论 |
| **LMDirichlet** | 短文档/短查询 | 适合稀疏文本 |
| **LMJelinekMercer** | 长查询文本 | 适合冗长查询 |

> 大多数场景下 BM25 已经是最优选择，只有在明确了解评分需求并经过充分测试后，才建议切换为其他算法。
## 注意事项

- `similarity` 在索引创建后可以通过关闭索引 → 更新设置 → 重新打开的方式修改
- 更改 similarity 后，已索引的文档不会自动重新评分，需要重建索引才能完全生效


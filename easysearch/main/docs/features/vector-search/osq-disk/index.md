---
title: "OSQ Disk 搜索"
date: 0001-01-01
description: "Easysearch 3.0 磁盘量化向量索引的映射、查询、重排和迁移说明。"
summary: "OSQ Disk 搜索 #  Easysearch 3.0 内置 OSQ Disk，通过量化向量和磁盘簇检索生成候选，再按配置使用保留的原精度向量重排。无需安装 k-NN 插件。
对外使用兼容 API 名称 index_options.type: &quot;bbq_disk&quot;，不接受 osq_disk 别名。 OSQ Disk 与 OSQ HNSW 是不同的索引类型；Disk 不使用 HNSW 的 m 和 ef_construction 参数。 省略 index_options 时仍使用普通 HNSW，不会自动选择 Disk；普通 HNSW 检索字段仍需显式 dims 和 index: true。
创建索引与写入 #  集群所有节点应先升级到支持 OSQ Disk 的 Easysearch 3.0 版本，再创建 Disk 字段。
PUT /disk-vectors { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;category&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;vector&#34;: { &#34;type&#34;: &#34;dense_vector&#34;, &#34;dims&#34;: 4, &#34;element_type&#34;: &#34;float&#34;, &#34;similarity&#34;: &#34;l2_norm&#34;, &#34;index_options&#34;: { &#34;type&#34;: &#34;bbq_disk&#34;, &#34;cluster_size&#34;: 384, &#34;rescore_vector&#34;: { &#34;oversample&#34;: 3 } } } } } } PUT /disk-vectors/_doc/1?"
---


# OSQ Disk 搜索

Easysearch 3.0 内置 **OSQ Disk**，通过量化向量和磁盘簇检索生成候选，再按配置使用保留的原精度向量重排。无需安装 k-NN 插件。

对外使用兼容 API 名称 `index_options.type: "bbq_disk"`，不接受 `osq_disk` 别名。
OSQ Disk 与 [OSQ HNSW](./osq-hnsw.md) 是不同的索引类型；Disk 不使用 HNSW 的 `m` 和 `ef_construction` 参数。
省略 `index_options` 时仍使用普通 HNSW，不会自动选择 Disk；普通 HNSW 检索字段仍需显式 `dims` 和 `index: true`。

## 创建索引与写入

集群所有节点应先升级到支持 OSQ Disk 的 Easysearch 3.0 版本，再创建 Disk 字段。

```json
PUT /disk-vectors
{
  "mappings": {
    "properties": {
      "category": { "type": "keyword" },
      "vector": {
        "type": "dense_vector",
        "dims": 4,
        "element_type": "float",
        "similarity": "l2_norm",
        "index_options": {
          "type": "bbq_disk",
          "cluster_size": 384,
          "rescore_vector": { "oversample": 3 }
        }
      }
    }
  }
}
```

```json
PUT /disk-vectors/_doc/1?refresh=true
{
  "category": "guide",
  "vector": [1.0, 2.0, 3.0, 4.0]
}
```

向量必须为单值数组，分量为有限数，长度与字段维度一致。`bfloat16` 字段按所声明的精度保留原向量；需要保留输入数组的原始 JSON 精度时，应在建索引时显式关闭向量的自动 source 排除。

## 映射参数

| 参数 | 默认值 | 约束与含义 |
| --- | --- | --- |
| `dims` | 可由首个有效向量确定 | 1–4096；生产环境建议显式指定 |
| `element_type` | `float` | 支持 `float`、`bfloat16`；不支持 `byte`、`bit` |
| `index` | `true` | 显式 `bbq_disk` 不接受 `false` |
| `similarity` | `cosine` | 也支持 `dot_product`、`l2_norm`、`max_inner_product`，输入须满足对应相似度的约束 |
| `index_options.bits` | `1` | 当前仅支持 1-bit 索引量化 |
| `index_options.cluster_size` | `384` | 64–65536的整数，控制构建时的目标簇大小 |
| `index_options.flat_index_threshold` | `-1` | `-1` 使用 `3 × cluster_size`；`0` 禁用小段平面检索；正整数指定阈值 |
| `index_options.default_visit_percentage` | `0` | 有限数，范围0–100；`0` 为自动确定访问目标 |
| `index_options.rescore_vector.oversample` | `3` | `0` 关闭原精度重排；非零范围为[1,10] |
| `index_options.on_disk_rescore` | `false` | 原向量读取策略；两种取值均保留原向量，实际IO方式取决于平台与文件布局 |
| `index_options.precondition` | `false` | 当前不支持 `true` |
| `index_options.auto_calibrate` | `false` | 当前不支持 `true` |

维度、元素类型和相似度不能原地修改。Disk 与普通 HNSW、OSQ HNSW 之间不能通过更新同一字段的 mapping 转换；应创建目标索引并 Reindex。
更新允许调整的构建参数只影响后续生成的段，不会立即重写已有段。

## 查询与过滤

顶层 `knn` 用于协调全局 top-k：

```json
POST /disk-vectors/_search
{
  "size": 10,
  "_source": false,
  "knn": {
    "field": "vector",
    "query_vector": [1.0, 2.0, 3.0, 4.0],
    "k": 10,
    "num_candidates": 100,
    "filter": { "term": { "category": "guide" } },
    "rescore_vector": { "oversample": 3 }
  }
}
```

也可以使用 DSL `query.knn` 与其它查询组合：

```json
POST /disk-vectors/_search
{
  "size": 10,
  "query": {
    "knn": {
      "field": "vector",
      "query_vector": [1.0, 2.0, 3.0, 4.0],
      "k": 10,
      "num_candidates": 100,
      "filter": { "term": { "category": "guide" } }
    }
  }
}
```

`k` 必须为正整数，`num_candidates` 不小于 `k` 且不超过10000。建议显式设置两者，避免不同入口的默认值影响结果数量。
查询级 `rescore_vector.oversample` 覆盖 mapping 配置，接受 `0` 或不小于1的有限数；重排候选池受候选上限约束。
开启重排时会按需要扩大有效候选预算，以容纳重排池。

可在 `knn` 内指定 `visit_percentage` 覆盖字段的 `default_visit_percentage`，范围为0–100。
`0` 表示自动确定访问目标；正值控制每段的访问目标比例。它是近似检索的访问目标，不保证只读取某个固定比例的文档；过滤、候选补足等条件可能增加访问工作。

将希望参与候选选择的条件放在 `knn.filter` 中。`post_filter` 在候选选择后执行，可能使最终结果少于 `k`。
访问目标、候选数量和重排倍率都可能影响召回率和查询成本，应按业务数据选择。

两种查询入口也可用lookup替代显式 `query_vector`，引用已refresh的向量。以下是knn对象内的参数片段：

```json
{
  "query_vector_builder": {
    "lookup": { "index": "disk-vectors", "id": "1", "path": "vector" }
  }
}
```

`query_vector` 与 `query_vector_builder` 必须且只能提供一个。lookup不是realtime GET；未刷新或不可搜索的文档不能当作有效查询向量。
`similarity` 阈值含义与字段相似度相关：cosine/dot_product为最小相似度，L2为最大距离，max_inner_product为最小内积。
完整度量与分数转换见 [原生HNSW得分计算](./native-hnsw.md#得分计算)，量化候选分数与raw重排分数需分别理解。

## Nested 与向量读取

Disk 支持 nested 向量检索和 `inner_hits`。外层结果按父文档去重；`inner_hits` 返回相应的子文档。
同时需要父、子响应恢复向量时，应在外层和 `inner_hits` 中都显式请求 `_source: {"exclude_vectors": false}`。

### Nested 完整示例

下面创建4维 L2 nested 字段，与本页普通字段示例使用相同维度：

```json
PUT /osq-disk-nested
{
  "mappings": {
    "properties": {
      "tenant": { "type": "keyword" },
      "children": {
        "type": "nested",
        "properties": {
          "tag": { "type": "keyword" },
          "vector": {
            "type": "dense_vector",
            "dims": 4,
            "similarity": "l2_norm",
            "index_options": { "type": "bbq_disk" }
          }
        }
      }
    }
  }
}
```

```json
PUT /osq-disk-nested/_doc/article-1?refresh=true
{
  "tenant": "allowed",
  "children": [
    { "tag": "keep", "vector": [0.25, 0.0, 0.0, 0.0] },
    { "tag": "keep", "vector": [2.0, 0.0, 0.0, 0.0] },
    { "tag": "drop", "vector": [0.0, 0.0, 0.0, 0.0] },
    { "tag": "keep" }
  ]
}
```

顶层入口使用 `knn.inner_hits`：

```json
POST /osq-disk-nested/_search
{
  "size": 2,
  "_source": { "exclude_vectors": false },
  "knn": {
    "field": "children.vector",
    "query_vector": [0.0, 0.0, 0.0, 0.0],
    "k": 2,
    "num_candidates": 10,
    "visit_percentage": 100,
    "rescore_vector": { "oversample": 3 },
    "filter": [
      { "term": { "tenant": "allowed" } },
      { "term": { "children.tag": "keep" } }
    ],
    "inner_hits": {
      "name": "selected",
      "size": 2,
      "sort": [{ "_score": "desc" }],
      "_source": { "exclude_vectors": false }
    }
  }
}
```

DSL 入口使用外层 `nested.inner_hits`，不放在 `query.knn` 中：

```json
POST /osq-disk-nested/_search
{
  "size": 2,
  "_source": { "exclude_vectors": false },
  "query": {
    "nested": {
      "path": "children",
      "score_mode": "max",
      "query": {
        "knn": {
          "field": "children.vector",
          "query_vector": [0.0, 0.0, 0.0, 0.0],
          "k": 2,
          "num_candidates": 10,
          "visit_percentage": 100,
          "rescore_vector": { "oversample": 3 },
          "filter": [
            { "term": { "tenant": "allowed" } },
            { "term": { "children.tag": "keep" } }
          ]
        }
      },
      "inner_hits": {
        "name": "selected",
        "size": 2,
        "sort": [{ "_score": "desc" }],
        "_source": { "exclude_vectors": false }
      }
    }
  }
}
```

两个请求都只返回父文档 `article-1`，父分数约为 `0.9411765`；子命中总数为2，`_nested.offset` 为0、1，分数约为 `0.9411765`、`0.2`。
父文档按符合根/子过滤条件的最佳子向量得分排序；没有向量的子文档不参与向量命中。
`inner_hits.from/size` 控制子分页，offset始终指原数组位置。例如 `from: 1, size: 1` 仅返回offset=1，总数仍为2。
本例使用全访问目标及raw重排方便说明分数，不代表生产场景固定使用100%访问；关闭重排后不能要求相同的raw分数。
inner_hits在匹配的子文档范围评分，不另起一次独立ANN召回。

### 取回向量

新建 OSQ 索引默认自动从物理 `_source` 排除向量，保留的原精度向量仍用于重排、字段读取及 source 恢复。
搜索需要返回向量时，可写为：

```json
POST /disk-vectors/_search
{
  "_source": { "exclude_vectors": false },
  "knn": {
    "field": "vector",
    "query_vector": [1.0, 2.0, 3.0, 4.0],
    "k": 10,
    "num_candidates": 100
  }
}
```

GET 可使用 `GET /disk-vectors/_doc/1?_source_exclude_vectors=false`。
也支持 `fields`、`docvalue_fields` 和向量脚本读取；恢复值遵循保留的 `float`／`bfloat16` 精度，不是从 1-bit 码反量化得到。
需要在物理 `_source` 中保留输入数组时，在建索引 settings 中设置 `index.mapping.exclude_source_vectors: false`；该设置不能动态修改。

cosine向量可能经历归一化及取回时的模长恢复，float恢复值也可能有舍入差异，不保证输入JSON或数值逐位不变。
自动排除关闭时，mapping中永久排除的向量不能依靠请求选项补回；显式禁用 `_source` 也不能靠该选项重建完整文档。
更多source过滤及字段读取说明见 [OSQ HNSW 的向量存储与取回](./osq-hnsw.md#向量存储与取回)。

### 持久化与恢复

Disk向量和nested关系使用正常索引持久化及同产品快照流程。flush后可close/open，快照恢复后仍需核对实际向量查询、inner_hits分数/offset和向量内容。
对只读索引执行forceMerge后，如需通过 `_segments` 检查搜索使用的段，应先refresh；commit段数与当前搜索reader的段数可能暂时不同。
不要仅凭健康状态或快照成功响应判定恢复内容正确。

## 迁移与适用边界

- 历史索引不会因升级程序自动变成 Disk。转换索引类型时，预先创建目标 mapping，再使用 [Reindex]({{< relref "/docs/operations/data-management/reindex.md" >}})。
- 只有支持相应能力的节点才能承载、查询 Disk 字段；不要将此类索引分配到旧版本节点。
- API 参数兼容不代表能够与 Elasticsearch 互读 segment 或跨产品恢复快照。迁移时复制文档并由目标索引重新构建。
- OSQ 为有损量化，不保证固定召回率，也不保证与其它引擎返回相同的 ANN 集合或逐位相同的近似分数。
- 索引还保存原向量、校正信息及其它必要数据，不能将1-bit码解释为整个索引缩小32倍。

原生 HNSW、OSQ HNSW 与 Disk 的接口选择见[向量搜索](./_index.md)。


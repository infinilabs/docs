---
title: "OSQ HNSW 搜索"
date: 0001-01-01
description: "Easysearch 3.0 OSQ HNSW 的映射、写入、查询、重排与向量存储恢复说明。"
summary: "OSQ HNSW 搜索 #  Easysearch 3.0 内置 OSQ HNSW 向量搜索，无需安装 k-NN 插件。本文适用于 Easysearch 3.0。
此功能属于 Easysearch OSQ 技术族。OSQ 指 Lucene 的 Optimized Scalar Quantization（优化的标量量化）；当前编码为索引 1-bit、查询 4-bit。量化降低候选检索数据的内存需求，同时带来近似误差，需要结合候选数量和原精度重排评估召回率。
对外沿用 ES 的兼容 API 名称。 mapping 使用 index_options.type: &quot;bbq_hnsw&quot;，不引入 osq_hnsw 类型或 API 别名；相关 settings、查询、写入和返回字段沿用兼容接口。
OSQ HNSW 对索引向量作 1-bit 量化，用量化分数搜索 HNSW 候选，再按配置使用保留的原精度向量重排。底层使用 Apache Lucene 10.5.2；REST API 兼容范围为 Elasticsearch 9.5.6 普通索引中的显式 bbq_hnsw 配置。
必须显式选择 bbq_hnsw。 省略 index_options 保留 Easysearch 原有普通 HNSW 选型，不按向量维度自动切换为 OSQ HNSW 或其它量化类型。本文不要求设置 index."
---


# OSQ HNSW 搜索

Easysearch 3.0 内置 **OSQ HNSW** 向量搜索，无需安装 k-NN 插件。本文适用于 Easysearch 3.0。

此功能属于 **Easysearch OSQ** 技术族。OSQ 指 Lucene 的 Optimized Scalar Quantization（优化的标量量化）；当前编码为索引 1-bit、查询 4-bit。量化降低候选检索数据的内存需求，同时带来近似误差，需要结合候选数量和原精度重排评估召回率。

**对外沿用 ES 的兼容 API 名称。** mapping 使用 `index_options.type: "bbq_hnsw"`，不引入 `osq_hnsw` 类型或 API 别名；相关 settings、查询、写入和返回字段沿用兼容接口。

OSQ HNSW 对索引向量作 1-bit 量化，用量化分数搜索 HNSW 候选，再按配置使用保留的原精度向量重排。底层使用 Apache Lucene 10.5.2；REST API 兼容范围为 Elasticsearch 9.5.6 普通索引中的显式 `bbq_hnsw` 配置。

**必须显式选择 `bbq_hnsw`。** 省略 `index_options` 保留 Easysearch 原有普通 HNSW 选型，不按向量维度自动切换为 OSQ HNSW 或其它量化类型。本文不要求设置 `index.mode`，也不表示 Easysearch 支持 ES 的其它索引模式。

使用前应将集群全部节点升级到具备 OSQ 能力的 3.0 版本；含不具备相应能力的节点时，创建或更新 OSQ 映射会被拒绝。历史索引不会因节点升级而自动变为 OSQ 索引；建议新建目标索引并 Reindex。

## 映射

```json
PUT /osq-demo
{
  "settings": {
    "index.mapping.exclude_source_vectors": true
  },
  "mappings": {
    "properties": {
      "title": { "type": "keyword" },
      "vector": {
        "type": "dense_vector",
        "dims": 1024,
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "bbq_hnsw",
          "m": 64,
          "ef_construction": 512
        }
      }
    }
  }
}
```

以下参数限定于具备相应能力的新3.0索引；历史索引和混合节点不能仅凭产品版本号获得相同默认语义。

| 参数 | 默认或约束 |
| --- | --- |
| `dims` | 64–4096；显式 OSQ HNSW 可在省略后由首个有效向量确定维度，生产配置建议明确指定 |
| `element_type` | `float`，也接受 `bfloat16`；显式 OSQ HNSW 不接受 `byte`、`bit` |
| `index` | 显式 OSQ HNSW 省略时为 `true`，不能与 `false` 组合 |
| `similarity` | `cosine`；也接受 `dot_product`、`l2_norm`、`max_inner_product` |
| `index_options.m` | 16，范围1–512 |
| `index_options.ef_construction` | 100，范围1–3200 |
| `index_options.rescore_vector.oversample` | 新语义索引默认为3；mapping 可显式设置0或[1,10]，空对象非法 |
| `index_options.on_disk_rescore` | `false`；开启后根据实际目录和段布局选择 raw 读取方式，不保证所有段均走 direct IO |
| `index_options.flat_index_threshold` | 未指定或任何负值按300处理；0禁用按小段阈值跳过构图的优化 |

向量必须是有限数值，长度与确定后的维度一致。单文档同一字段只接受一个向量；`null` 或缺失不生成向量索引项。cosine 不接受零向量；dot_product 要求单位向量。bfloat16 会降低保留值的精度，不承诺恢复客户端输入的全部小数位。

`m`、`ef_construction` 的范围由底层 format 构造时校验；非法值可能在首次创建字段 writer 时才失败，不应依赖创建 mapping 一定提前报错。

已有字段的维度、元素类型和相似度不能直接改变；`m` 只能保持或增大，`ef_construction` 可调整。mapping 更新不会立即重写全部旧段，不应把更新成功当成所有数据已使用新构图参数。普通 HNSW 向显式 OSQ HNSW 的转换也会暂时形成混合段，不支持由 OSQ HNSW 改回普通 HNSW。

## 写入

Index、Create 和 Bulk 都使用普通文档写入接口。以下脚本生成一条符合上面 1024 维映射的示例文档：

```bash
python3 - <<'PY' | curl -sS -X PUT \
  'http://localhost:9200/osq-demo/_doc/seed?refresh=true' \
  -H 'Content-Type: application/json' --data-binary @-
import json
print(json.dumps({"title": "example", "vector": [1.0] + [0.0] * 1023}))
PY
```

Update 和 Reindex 复用完整向量源恢复逻辑；自动省略响应中的向量，不代表局部更新或重建索引时应丢弃向量。显式关闭 `_source` 会影响依赖原文档内容的这些操作，不能将它与自动省略向量视为同一个开关。

Bulk 同样在普通文档中传入 `vector` 数组，无需客户端预先量化；向量转换和量化由服务端完成。批量写入后，先检查每个 item 的结果，再 refresh 后执行查询。

## 两种查询入口

可以直接在 `query_vector` 中提供1024维数值数组，也可以通过 `query_vector_builder.lookup` 引用已经 refresh 的向量。二者必须且只能提供一个。

顶层 `knn`：

```json
POST /osq-demo/_search
{
  "size": 10,
  "knn": {
    "field": "vector",
    "query_vector_builder": {
      "lookup": { "index": "osq-demo", "id": "seed", "path": "vector" }
    },
    "k": 10,
    "num_candidates": 100,
    "rescore_vector": { "oversample": 3 }
  }
}
```

DSL `query.knn`：

```json
POST /osq-demo/_search
{
  "size": 10,
  "query": {
    "knn": {
      "field": "vector",
      "query_vector_builder": {
        "lookup": { "index": "osq-demo", "id": "seed", "path": "vector" }
      },
      "k": 10,
      "num_candidates": 100,
      "rescore_vector": { "oversample": 3 }
    }
  }
}
```

lookup 的参数是 `index`、`id`、`path`，按需传 `routing`；查询读取的是可搜索的已刷新向量，不是 realtime GET。文档或字段尚不可见时返回 404，不能因 `_source` 中存在数组就视为 lookup 已可用。可选 AI 文本向量 builder 需要对应插件与 provider 配置；使用 OSQ HNSW 本身不要求接入模型服务。

若查询向量来自应用，可用以下示例替代 lookup。示例与前面的 1024 维 mapping 一致，并在候选检索时过滤 `title`：

```bash
python3 - <<'PY' | curl -sS -X POST \
  'http://localhost:9200/osq-demo/_search' \
  -H 'Content-Type: application/json' --data-binary @-
import json
print(json.dumps({
    "size": 10,
    "knn": {
        "field": "vector",
        "query_vector": [1.0] + [0.0] * 1023,
        "k": 10,
        "num_candidates": 100,
        "filter": {"term": {"title": "example"}},
        "rescore_vector": {"oversample": 3}
    }
}))
PY
```

上述 curl 示例假设本地 HTTP 服务；启用安全与 TLS 时，使用实际 HTTPS 地址、证书和认证参数。

两种入口省略 `k` 的规则不同：

- 顶层取请求 `size` 作为有效k，`num_candidates` 必须不小于它。`size=10`、省略k、`num_candidates=5` 会拒绝。
- DSL 省略k时取 `min(size, num_candidates)`，上述配置的有效k为5。
- 显式k时两入口都要求 `num_candidates >= k`；候选数不得超过10000。

## 候选和重排

查询中的 `rescore_vector.oversample` 覆盖 mapping；0关闭 raw 重排，非零值须为有限数且至少为1。查询倍率没有mapping的10倍上限，实际重排窗口有上限：

```text
R = min(ceil(k × oversample), 10000)
C = max(num_candidates, R)
```

先按量化分数生成候选，再对R个候选读取raw并取最终k。`num_candidates` 是收集队列宽度，不是图节点访问数硬上限，也不是保证全部候选都做raw重排。例如k10/C1000/默认3只重排30个；倍率提高到R>C时还会抬高有效C。

重排只能重新排序进入窗口的候选，不能找回候选阶段已经遗漏的邻居。提高倍率不保证达到某个召回目标；请使用自己的精确真值评估，不用两端ANN结果逐条相同作为API兼容标准。

预过滤放在 `knn.filter` 中。外层 bool 过滤不能一概当成候选预过滤；普通请求不自动启用其它 semantic pipeline 的过滤规则。`similarity` 阈值在对应评分域应用，开启重排时按raw分数处理。查询取消、超时和权限限制继续生效。

nested 向量先按父文档去重。顶层 `knn.inner_hits` 用于nested子文档；DSL 入口在外层 `nested` 声明 `inner_hits`。inner hits 在匹配的子文档范围精确评分，不再执行一次ANN。非nested字段上的合法顶层inner_hits对象会被忽略。

## 向量存储与取回

新建 OSQ HNSW 索引的 `index.mapping.exclude_source_vectors` 默认开启，是建索引时确定的不可动态修改设置。它控制物理 `_source` 是否重复保存向量；仅升级节点或向旧索引新增 OSQ 字段不会自动切换旧索引的 source 语义。

向量分别用于以下存储与读取路径：

| 数据 | 用途 |
| --- | --- |
| 1-bit 量化码及校正数据 | 量化候选评分，配合 HNSW 图检索 |
| Lucene 段内保留的原精度向量（raw） | 原精度重排、向量字段读取和 source 恢复；始终保留 |
| 物理 `_source` 中的向量数组 | 自动排除关闭时按 source 配置保存；默认开启时省略这份重复存储 |

开启重排、字段读取和向量恢复使用 raw，**不是从 1-bit 数据反量化恢复**。`on_disk_rescore` 只影响 raw 的读取策略，设为 `false` 也仍然保存 raw，并不表示将整份 raw 固定加载到堆内存。

搜索默认省略向量；需要返回时可以使用：

```json
POST /osq-demo/_search
{
  "size": 1,
  "_source": { "exclude_vectors": false },
  "query": { "match_all": {} }
}
```

GET、GET source、mget 和search 的 URL 入口接受 `_source_exclude_vectors=false`。mget 可在每个文档的 `_source` 对象中覆盖默认取回选项；msearch 在每个子搜索的body中指定 `_source.exclude_vectors`，不要将该URL参数直接加到msearch。

```text
GET /osq-demo/_doc/seed?_source_exclude_vectors=false
GET /osq-demo/_source/seed?_source_exclude_vectors=false
```

若需要在物理 `_source` 中保留输入数组，应在创建索引时设置 `index.mapping.exclude_source_vectors: false`。
这与 mapping 中手工配置 `_source.excludes: ["vector"]` 不同：后者是存储过滤规则。
自动排除关闭时，被该存储过滤规则永久移除的数组不会通过 `exclude_vectors: false` 补回；需要后续 Update 或 Reindex 时，应先确认完整源可恢复。

`fields`、`docvalue_fields` 与脚本的向量读取是各自的字段接口，`_source: false` 不等于禁止合法字段读取；仍受字段权限限制。自动省略向量与请求级includes/excludes、字段权限及mapping存储过滤不是同一个层次。自动省略启用时，mapping过滤不会移除indexed向量的raw恢复来源；请求投影和权限仍应遵守。

例如，只返回向量字段而不返回 `_source`：

```json
POST /osq-demo/_search
{
  "size": 1,
  "_source": false,
  "docvalue_fields": [{ "field": "vector", "format": "array" }],
  "query": { "match_all": {} }
}
```

恢复值取决于保留的向量精度。float32通常不能保留原JSON所有双精度小数；bfloat16还会进一步舍入。显式关闭 `_source` 时，不能用 `exclude_vectors: false` 重新生成完整文档。若关闭自动省略并依赖mapping永久排除向量，也不能承诺source取回会补回被永久丢弃的内容。

## 索引级查询设置

除 mapping 中的构图和重排配置外，还支持以下动态索引设置：

| 设置 | 默认值 | 说明 |
| --- | --- | --- |
| `index.dense_vector.hnsw_filter_heuristic` | `ACORN` | 过滤搜索策略，也可设为 `FANOUT` |
| `index.dense_vector.hnsw_enable_early_termination` | `true` | 是否启用自适应早停 |

例如，显式设置策略并关闭早停以评估业务数据上的效果：

```json
PUT /osq-demo/_settings
{
  "index.dense_vector.hnsw_filter_heuristic": "ACORN",
  "index.dense_vector.hnsw_enable_early_termination": false
}
```

调整只影响后续查询，不重建图，也不保证召回提升。`index.mapping.exclude_source_vectors` 是建索引设置，不能通过该接口动态修改。

## 迁移与兼容边界

从 Elasticsearch 9 迁移时，在 Easysearch 3.0 中预先创建目标 mapping 和 settings，再使用 Remote Reindex 复制文档。
远端自动排除 `_source` 向量时，迁移读取路径会请求恢复向量；前提是远端仍保留可恢复的原精度向量且 `_source` 可用。
操作入口见[重建索引]({{< relref "/docs/operations/data-management/reindex.md" >}})。Reindex 不会复制源索引的 mapping、settings 或 HNSW 图。

迁移后应分别核对文档、ID、routing、向量值，以及代表性查询的召回和评分。两端独立构图、量化和候选检索可能产生不同的 ANN 命中集合；API 兼容不保证查询结果逐条相同，也不保证浮点分数逐位相等。

OSQ 是有损量化，召回率取决于数据分布、候选数量和重排配置，应使用业务数据评估。1-bit 是量化码的位宽，实际还包括校正数据、HNSW 图、raw、其它字段和副本，不能将其解释为整个索引体积缩小 32 倍。

Easysearch 与 ES 的 codec 身份、量化实现及 segment 文件布局不同；相同 REST mapping 不代表可以直接互读 segment 或跨产品恢复快照。OSQ HNSW 暂不支持 `bbq_disk` API，其它量化索引类型和旧插件查询语法不在本页范围内。


---
title: "原生 HNSW 搜索"
date: 0001-01-01
description: "使用 Easysearch 2.4.0 原生 dense_vector、knn query 和顶层 knn 执行 HNSW 向量搜索。"
summary: "原生 HNSW 搜索 #  Easysearch 2.4.0 内置基于 Lucene 的 HNSW 向量索引。新建原生 HNSW 索引不需要安装插件，使用 dense_vector mapping 和 标准 _search API 即可完成写入和检索。
旧 k-NN 插件继续使用 knn_dense_float_vector、knn_sparse_bool_vector 和 knn_nearest_neighbors。旧接口的使用方法参考 旧 k-NN 查询 API。不要在同一个字段上混用两套语法。
完整示例 #  创建索引 #  PUT /native-hnsw-demo { &#34;settings&#34;: { &#34;number_of_shards&#34;: 1, &#34;number_of_replicas&#34;: 0 }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;tenant&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;dense_vector&#34;, &#34;dims&#34;: 4, &#34;element_type&#34;: &#34;float&#34;, &#34;index&#34;: true, &#34;similarity&#34;: &#34;cosine&#34;, &#34;index_options&#34;: { &#34;type&#34;: &#34;hnsw&#34;, &#34;m&#34;: 16, &#34;ef_construction&#34;: 100 } } } } } 完整的 mapping 参数参考 dense_vector 字段类型。"
---


# 原生 HNSW 搜索

Easysearch 2.4.0 内置基于 Lucene 的 HNSW 向量索引。新建原生 HNSW 索引不需要安装插件，使用 `dense_vector` mapping 和
标准 `_search` API 即可完成写入和检索。

旧 k-NN 插件继续使用 `knn_dense_float_vector`、`knn_sparse_bool_vector` 和 `knn_nearest_neighbors`。旧接口的使用方法参考
[旧 k-NN 查询 API]({{< relref "/docs/features/vector-search/knn_api.md" >}})。不要在同一个字段上混用两套语法。

## 完整示例

### 创建索引

```json
PUT /native-hnsw-demo
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "tenant": {
        "type": "keyword"
      },
      "embedding": {
        "type": "dense_vector",
        "dims": 4,
        "element_type": "float",
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "hnsw",
          "m": 16,
          "ef_construction": 100
        }
      }
    }
  }
}
```

完整的 mapping 参数参考
[dense_vector 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/dense-vector.md" >}})。

### 批量写入

```json
POST /_bulk?refresh=true
{ "index": { "_index": "native-hnsw-demo", "_id": "v01" } }
{ "title": "向量文档一", "tenant": "a", "embedding": [1.0, 0.0, 0.0, 0.0] }
{ "index": { "_index": "native-hnsw-demo", "_id": "v02" } }
{ "title": "向量文档二", "tenant": "a", "embedding": [0.9, 0.1, 0.0, 0.0] }
{ "index": { "_index": "native-hnsw-demo", "_id": "v03" } }
{ "title": "向量文档三", "tenant": "b", "embedding": [0.7, 0.7, 0.0, 0.0] }
{ "index": { "_index": "native-hnsw-demo", "_id": "v04" } }
{ "title": "向量文档四", "tenant": "b", "embedding": [0.0, 1.0, 0.0, 0.0] }
```

Bulk 请求使用 NDJSON，每行必须以换行符结束。

## query-level knn

将 `knn` 放在 `query` 中，可以单独执行，也可以嵌入受支持的复合查询。

```json
POST /native-hnsw-demo/_search
{
  "size": 3,
  "query": {
    "knn": {
      "field": "embedding",
      "query_vector": [1.0, 0.0, 0.0, 0.0],
      "k": 3,
      "num_candidates": 4
    }
  }
}
```

### 查询参数

| 参数 | 是否必填 | 说明 |
| --- | --- | --- |
| `field` | 是 | 已索引的 `dense_vector` 字段名 |
| `query_vector` | 是 | 查询向量，维度必须与字段 mapping 一致 |
| `k` | 否 | 每个分片返回的最近邻数量，必须大于 0；省略时以请求的 `size` 为基准；仅在省略 `k` 时，如果同时设置了更小的 `num_candidates`，有效值会被限制为 `num_candidates` |
| `num_candidates` | 否 | 每个分片搜索的候选数量，必须不小于 `k`，最大为 10000 |
| `filter` | 否 | 在 HNSW 搜索过程中应用的预过滤查询，可传单个查询或查询数组 |
| `similarity` | 否 | 最低相似度阈值，含义取决于字段 mapping 的 `similarity` |
| `boost` | 否 | 对向量得分应用的权重 |
| `_name` | 否 | 命名查询，命中结果会返回对应的 `matched_queries` |

省略 `num_candidates` 时，其值按 `min(1.5 × k, 10000)` 计算并四舍五入。省略 `k` 时先以请求的 `size` 作为 `k`；如果显式设置的 `num_candidates` 小于该值，`k` 和候选数都会使用这个较小值。增大候选数量通常可以提高召回率，但会增加查询开销。

query-level `k` 是每个分片的候选数量，最终响应条数仍由外层 `size` 控制。需要跨分片严格执行全局 `k` 时，使用顶层 `knn`。

### 带过滤条件

`filter` 是向量搜索的预过滤条件，而不是在召回完成后再删除结果：

```json
POST /native-hnsw-demo/_search
{
  "size": 2,
  "query": {
    "knn": {
      "field": "embedding",
      "query_vector": [1.0, 0.0, 0.0, 0.0],
      "k": 2,
      "num_candidates": 4,
      "filter": {
        "term": {
          "tenant": "a"
        }
      }
    }
  }
}
```

`filter` 也可以是查询数组，数组中的条件全部作为过滤条件执行。

### 与复合查询组合

query-level `knn` 可以用于 `bool`、`constant_score`、`dis_max`、`boosting` 和 `function_score` 等核心复合查询。以下示例要求
文档属于租户 `a`，并使用向量相似度排序：

```json
POST /native-hnsw-demo/_search
{
  "size": 2,
  "query": {
    "bool": {
      "must": {
        "knn": {
          "field": "embedding",
          "query_vector": [1.0, 0.0, 0.0, 0.0],
          "k": 2,
          "num_candidates": 4
        }
      },
      "filter": {
        "term": {
          "tenant": "a"
        }
      }
    }
  }
}
```

## 顶层 knn

将 `knn` 放在搜索请求顶层，可以在多分片搜索中协调候选结果并返回全局 top-k。

> Easysearch 2.4.0 不支持在同一个请求中同时使用顶层 `knn` 和普通 `query`。需要组合 lexical 查询和向量查询时，
> 请将 query-level `knn` 放入 `bool` 等复合查询；不要把 `query` 与顶层 `knn` 并列发送。

例如，以下请求会失败：

```json
POST /native-hnsw-demo/_search
{
  "query": {
    "match": {
      "title": "向量"
    }
  },
  "knn": {
    "field": "embedding",
    "query_vector": [1.0, 0.0, 0.0, 0.0],
    "k": 3
  }
}
```

错误响应的 `reason` 为：

```text
P0 does not support combining a lexical [query] with top-level [knn]
```

### 正确的顶层 knn

```json
POST /native-hnsw-demo/_search
{
  "size": 3,
  "knn": {
    "field": "embedding",
    "query_vector": [1.0, 0.0, 0.0, 0.0],
    "k": 3,
    "num_candidates": 4
  }
}
```

顶层 `knn` 使用与 query-level `knn` 相同的参数，但 `k` 表示协调后的全局最近邻数量。Easysearch 2.4.0 支持单个顶层
`knn` 对象，也接受只包含一个对象的数组：

```json
POST /native-hnsw-demo/_search
{
  "size": 3,
  "knn": [
    {
      "field": "embedding",
      "query_vector": [1.0, 0.0, 0.0, 0.0],
      "k": 3,
      "num_candidates": 4
    }
  ]
}
```

数组中不能包含多个 `knn` 对象。多路 kNN，以及在同一请求中同时使用顶层 `knn` 和 lexical `query`，不属于 2.4.0 的原生 HNSW 支持范围。

## 相似度阈值

查询参数 `similarity` 使用 mapping 中相似度函数的原始度量空间：

- `cosine` 和 `dot_product` 使用最小相似度；
- `l2_norm` 使用最大允许距离；
- `max_inner_product` 使用最小内积。

例如，只返回 cosine similarity 不低于 `0.8` 的候选结果：

```json
POST /native-hnsw-demo/_search
{
  "size": 10,
  "query": {
    "knn": {
      "field": "embedding",
      "query_vector": [1.0, 0.0, 0.0, 0.0],
      "k": 10,
      "num_candidates": 100,
      "similarity": 0.8
    }
  }
}
```

阈值可能使最终命中数量少于 `k` 或 `size`。

## 得分计算

响应中的 `_score` 是由 mapping 选择的原始相似度或距离转换得到的正数。转换规则如下：

| mapping `similarity` | 原始度量 | 未应用 `boost` 的 `_score` |
| --- | --- | --- |
| `cosine` | 余弦相似度 `s` | `max((1 + s) / 2, 0)` |
| `dot_product` | 单位向量点积 `s` | `max((1 + s) / 2, 0)` |
| `l2_norm` | 欧氏距离 `d` | `1 / (1 + d²)` |
| `max_inner_product` | 内积 `s < 0` | `1 / (1 - s)` |
| `max_inner_product` | 内积 `s >= 0` | `s + 1` |

对于单独的 `knn` 子句，该子句贡献的分数等于转换后的分数乘以查询的 `boost`；放入复合查询后，最终 `_score` 还会按
复合查询规则组合其他子句的分数。`similarity` 阈值始终在原始度量空间中判断，并且在应用 `boost` 之前完成。因此，提高
`boost` 只影响命中后的排序分数，不会让低于阈值的向量通过过滤。

使用本页示例数据和 cosine 查询时，响应的前三个命中如下：

```json
{
  "hits": {
    "hits": [
      { "_id": "v01", "_score": 1.0 },
      { "_id": "v02", "_score": 0.9969419 },
      { "_id": "v03", "_score": 0.8535534 }
    ]
  }
}
```

浮点计算和 HNSW 近似搜索可能产生很小的分数差异，不要依赖 `_score` 的字符串完全相等。

## 选择查询形式

| 场景 | 推荐形式 |
| --- | --- |
| 单分片索引或需要嵌入复合查询 | query-level `knn` |
| 多分片索引并要求严格全局 top-k | 顶层 `knn` |
| 需要在 HNSW 搜索中预过滤 | 两种形式均可使用 `filter` |
| 多路 kNN，或顶层 `knn` 与 lexical `query` 并列 | 2.4.0 暂不支持 |

## 生产调优

原生 HNSW 的召回率、查询成本、写入成本和存储占用由 mapping 参数、查询参数和 segment 拓扑共同决定。先使用代表真实业务的
向量和查询集建立 Recall 与延迟基线，再逐项调整参数；不要只依据单次查询耗时调优。

| 参数 | 增大后的通常影响 | 调整建议 |
| --- | --- | --- |
| `m` | 提高图连通性和 Recall，同时增加构图 CPU、内存和存储 | 建索引前确定；更新后只影响新 segment，减小需要新索引 |
| `ef_construction` | 提高新图质量，同时降低写入吞吐并增加构图开销 | 写入吞吐和 Recall 一起验证；更新只影响新 segment |
| `num_candidates` | 通常提高 Recall，同时增加每次查询的 CPU 和延迟 | 从 `k` 的数倍开始，用真实查询集逐步增加 |
| `k` | 返回更多邻居，也会增加分片候选和协调成本 | 只设置业务真正需要的邻居数量 |

### 写入建议

- 使用 Bulk 写入，检查每个 Bulk item 的状态，不能只检查 HTTP 状态码。
- 大批量初始导入时可适当延长 `refresh_interval`，导入完成后恢复生产值并执行一次 refresh。
- 在业务允许且已有其他数据保护措施时，可在初始导入期间暂时将副本数设为 `0`；验收完成后恢复副本并等待集群健康。
- 避免大量小 Bulk 和过于频繁的 refresh。每次 refresh 都可能产生新 segment，而每个 segment 都有独立的 HNSW 图。
- 同时观察写入吞吐、merge 时间、JVM/进程内存和磁盘增长；HNSW 构图成本不应只用 Bulk 请求延迟判断。

### Segment 与 merge

HNSW 图按 segment 构建。查询需要访问 shard 中所有相关 segment，再合并各 segment 的候选结果。segment 数量会直接影响查询
CPU 和延迟，后台 merge 则会消耗 CPU、I/O 和额外临时磁盘空间。

Easysearch 2.4.0 尚未提供与 Elasticsearch 8.19 对齐的 shard 内 segment 并行搜索。自然多 segment 索引上的查询吞吐可能低于
单 segment 索引，也可能低于相同合同下的 Elasticsearch 8.19。上线前必须使用生产预期的 segment 拓扑进行压测，不能用
force merge 后的单 segment 结果代表持续写入场景。

force merge 适合不再写入的只读索引维护，不应作为活跃写入索引的日常性能手段。它可能产生很大的 segment，并在执行期间消耗
大量 CPU、I/O 和临时磁盘。优先通过合理的 Bulk、refresh、分片大小和生命周期策略控制 segment 数量。

### 容量与监控

容量评估至少要包含原始 `_source`、浮点向量、HNSW 图、其他字段、segment 合并临时空间和所有副本。不同维度、`m`、文档数和
数据分布的占用差异很大，应使用真实数据实测 store size、进程 RSS 和文件系统缓存需求。

常用只读检查：

```text
GET /native-hnsw-demo/_segments
GET /native-hnsw-demo/_stats/store,indexing,merges,refresh
GET /_nodes/stats/jvm,process,fs,thread_pool
```

重点关注 segment 数持续增长、merge backlog、磁盘水位、JVM GC、进程 RSS、search/write thread pool rejection，以及 Recall、
查询 P50/P95/P99 和写入吞吐的长期变化。Easysearch 2.4.0 没有单独的 HNSW 图内存统计项，不能只看 JVM heap 判断容量。

### 常见问题

| 现象 | 优先检查 | 处理方向 |
| --- | --- | --- |
| Recall 不足 | `num_candidates`、`m`、`ef_construction`、过滤条件和评测真值 | 先增加 `num_candidates`；仍不足时用新索引验证构图参数 |
| 查询延迟或 CPU 高 | segment 数、候选数、并发、过滤选择性和后台 merge | 降低无效候选；控制 refresh 和 segment；隔离写入/查询基准 |
| 写入吞吐下降 | Bulk 大小、refresh、副本、merge、GC、磁盘 I/O | 减少小 Bulk 和频繁 refresh；排查 merge/I/O，而不是降低正确性校验 |
| store 增长过快 | 维度、`m`、副本、segment 和 `_source` | 用同合同 A/B 测量；调整参数需重新评估 Recall |
| 命中少于 `k` | `similarity` 阈值、filter、可用文档数 | 放宽阈值或过滤条件；确认符合条件的文档数 |
| 写入被拒绝 | 维度、有限数值、cosine 零向量、dot product 单位长度 | 根据具体错误修正向量生成和归一化流程 |

## 迁移现有索引

不能通过更新旧索引 mapping 把旧 k-NN 字段或其他字段原地转换成原生 `dense_vector`。以下流程使用新索引、Reindex 和原子 alias
切换；前提是源 `_source` 中的向量已经是与目标 `dims` 一致的 JSON 浮点数组。

1. 创建带有目标 `dense_vector` mapping 的新索引，例如 `products-vector-v2`。
2. 从当前索引 Reindex 到新索引：

```json
POST /_reindex?wait_for_completion=false
{
  "source": {
    "index": "products-vector-v1"
  },
  "dest": {
    "index": "products-vector-v2"
  }
}
```

3. 检查任务结果和 Bulk failure，核对文档数、mapping、代表性查询、Recall、错误率和资源占用。文档数相同不能替代查询验收。
4. 在短暂停写窗口内同步增量数据，或提前使用双写保证两边一致。
5. 如果应用通过 `products-vector` alias 访问索引，使用一个 `_aliases` 请求完成原子切换：

```json
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "products-vector-v1",
        "alias": "products-vector"
      }
    },
    {
      "add": {
        "index": "products-vector-v2",
        "alias": "products-vector",
        "is_write_index": true
      }
    }
  ]
}
```

6. 保留旧索引一段观察期。需要回滚时，用相反的 alias 操作切回，并处理切换后写入新索引的增量数据。确认不再回滚后再删除旧索引。

如果应用直接使用物理索引名，先改为 alias 或在应用配置层完成切换。迁移前应创建可恢复的快照，并预留新旧索引与 merge 所需的
磁盘空间。完整 API 参考[重建索引]({{< relref "/docs/operations/data-management/reindex.md" >}})和
[别名]({{< relref "/docs/operations/data-management/aliases.md" >}})。

## Elasticsearch 8.19 客户端

Easysearch 2.4.0 已使用 Elasticsearch Java `8.19.17` 和 Python `8.19.3` 官方客户端验证原生 HNSW mapping、Bulk、
query-level kNN、顶层 kNN、认证和错误响应。节点需要开启 Elasticsearch API 兼容模式：

```yaml
elasticsearch.api_compatibility: true
elasticsearch.api_compatibility_version: "8.19.17"
```

这些是节点启动配置，默认分别为 `false` 和 `7.10.2`，不能通过集群设置动态修改。修改后按部署流程重启节点，并先通过客户端的
集群信息请求验证连接到了预期集群。完整默认值、生效方式和升级说明见
[Elasticsearch 兼容性配置]({{< relref "/docs/deployment/config/configuration_file.md#elasticsearch-兼容性" >}})。

### Python 8.19

```python
from elasticsearch import Elasticsearch

client = Elasticsearch(
    "https://easysearch.example.com:9200",
    basic_auth=("admin", "<password>"),
    ca_certs="/path/to/http-ca.crt",
)

response = client.search(
    index="native-hnsw-demo",
    size=3,
    query={
        "knn": {
            "field": "embedding",
            "query_vector": [1.0, 0.0, 0.0, 0.0],
            "k": 3,
            "num_candidates": 100,
        }
    },
)

for hit in response["hits"]["hits"]:
    print(hit["_id"], hit["_score"])
```

### Java 8.19

依赖版本应保持一致：

```text
co.elastic.clients:elasticsearch-java:8.19.17
org.elasticsearch.client:elasticsearch-rest-client:8.19.17
```

以下示例使用 query-level kNN；生产环境还需要在 `RestClient` 上配置认证和 CA 信任：

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.KnnQuery;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;

import java.util.List;
import java.util.Map;

try (RestClientTransport transport = new RestClientTransport(
    RestClient.builder(HttpHost.create("https://easysearch.example.com:9200")).build(),
    new JacksonJsonpMapper()
)) {
    ElasticsearchClient client = new ElasticsearchClient(transport);
    KnnQuery knn = KnnQuery.of(k -> k
        .field("embedding")
        .queryVector(List.of(1.0f, 0.0f, 0.0f, 0.0f))
        .k(3)
        .numCandidates(100)
    );

    SearchResponse<Map> response = client.search(s -> s
        .index("native-hnsw-demo")
        .size(3)
        .query(q -> q.knn(knn)), Map.class);
}
```

客户端会发送 Elasticsearch 8.x vendor media type。不要手工删除或改写兼容请求头；连接失败时先检查节点兼容配置、TLS 信任、
认证信息和响应中的产品标识。

## 当前不支持的接口

Easysearch 2.4.0 原生 HNSW 不提供以下 Elasticsearch 向量接口：

- 已弃用的 `/_knn_search`；
- `knn` retriever；
- 多个顶层 `knn` 子句；
- `sub_searches` 和 `rank`；
- `_source.exclude_vectors`；
- nested kNN 和 `inner_hits`；
- byte、bit 和量化向量查询。

这些限制只针对本页的原生 HNSW。旧 k-NN 插件能力和语法继续按旧插件文档使用。


---
title: "_source 与字段存储"
date: 0001-01-01
description: "_source、doc_values 与字段检索策略的权衡。"
summary: "_source 与字段存储 #  _source 是文档写入时的原始 JSON，而字段的存储和检索有多种方式。本页讨论如何在&quot;读取灵活性、存储成本与性能&quot;之间做权衡。
_source：默认的真相来源 #  _source 保存了写入时的完整 JSON 文档。在 _source 开启的情况下：
 GET 请求可以直接返回原始文档 update 操作可以在服务端基于 _source 进行部分更新 需要重建索引时，可以直接从 _source 读取数据（参见 重建索引）  大多数场景下，建议保持 _source 开启，除非有非常严格的存储或合规要求。
_source 字段过滤 #  在查询时，可以通过 _source_includes 和 _source_excludes 只返回需要的字段，减少网络传输：
// GET 请求中过滤 _source GET /my-index/_doc/1?_source_includes=title,date // Search 请求中过滤 _source POST /my-index/_search { &#34;_source&#34;: { &#34;includes&#34;: [&#34;title&#34;, &#34;date&#34;], &#34;excludes&#34;: [&#34;description&#34;] }, &#34;query&#34;: { &#34;match_all&#34;: {} } } // 完全不返回 _source GET /my-index/_doc/1?"
---


# _source 与字段存储

`_source` 是文档写入时的原始 JSON，而字段的存储和检索有多种方式。本页讨论如何在"读取灵活性、存储成本与性能"之间做权衡。

## _source：默认的真相来源

`_source` 保存了写入时的完整 JSON 文档。在 `_source` 开启的情况下：

- GET 请求可以直接返回原始文档
- update 操作可以在服务端基于 `_source` 进行部分更新
- 需要重建索引时，可以直接从 `_source` 读取数据（参见 [重建索引]({{< relref "/docs/operations/data-management/reindex.md" >}})）

大多数场景下，**建议保持 `_source` 开启**，除非有非常严格的存储或合规要求。

### _source 字段过滤

在查询时，可以通过 `_source_includes` 和 `_source_excludes` 只返回需要的字段，减少网络传输：

```json
// GET 请求中过滤 _source
GET /my-index/_doc/1?_source_includes=title,date

// Search 请求中过滤 _source
POST /my-index/_search
{
  "_source": {
    "includes": ["title", "date"],
    "excludes": ["description"]
  },
  "query": { "match_all": {} }
}

// 完全不返回 _source
GET /my-index/_doc/1?_source=false
```

> 这是接口级别的"投影"，不影响存储——`_source` 中仍然保存完整文档。

## Doc Values：列式存储

Easysearch 中，大多数字段类型默认使用 `doc_values` 进行列式存储，这使得：

- **聚合、排序、脚本**等操作可以直接从 `doc_values` 读取，无需加载 `_source`
- 以列式存储，对聚合和排序操作非常高效
- 默认情况下，除了 `text` 和 `annotated_text` 字段外，所有字段都启用 `doc_values`

在查询中使用 `docvalue_fields` 可以直接从列式存储读取字段值：

```json
POST /my-index/_search
{
  "query": { "match_all": {} },
  "docvalue_fields": ["timestamp", "status_code"],
  "_source": false
}
```

如果某个字段永远不需要聚合或排序，可以在映射中关闭 `doc_values` 以节省磁盘空间：

```json
PUT /my-index
{
  "mappings": {
    "properties": {
      "description": {
        "type": "keyword",
        "doc_values": false
      }
    }
  }
}
```

## Stored Fields：按字段单独存储

> **注意**：`stored fields` 已经基本被 `doc_values` 替代。只有在极少数特殊场景下才需要使用。

`stored fields` 可以让你只从某些字段读取值，而不必返回整个 `_source`。但在现代用法中：

- 大多数场景下，直接依赖 `_source` + 客户端投影已经足够
- 对于聚合、排序等操作，`doc_values` 已经提供了更好的性能
- 只有在以下极端场景才考虑使用 `stored fields`：
  - `_source` 极大且不方便拆分
  - 读取路径非常敏感，对响应大小有严格要求
  - 需要返回的字段没有启用 `doc_values`（如 `text` 字段）

## 读取策略：按场景选择

| 场景 | 推荐策略 | 说明 |
|------|----------|------|
| 后台任务 / 管理工具 | 直接拉 `_source` | 处理灵活度最高 |
| 在线接口（延迟/带宽敏感） | `_source_includes` 限制字段 | 减少网络传输 |
| 聚合和排序 | `doc_values`（默认） | 无需加载 `_source`，列式读取更高效 |
| `text` 字段聚合 | 启用 `fielddata` | 注意内存开销，通常建议用 `keyword` 子字段替代 |
| 不需要 `_source` 的大批量扫描 | `_source: false` + `docvalue_fields` | 适合数据导出、ETL 场景 |

在设计接口与文档结构时，可以先规划好"接口级别的字段投影"，而不是在存储层做过早优化。

## 关闭 _source 的风险

在某些极端场景下，有人会考虑关闭 `_source` 以节省存储，但这会带来明显风险：

- 无法直接 GET 出完整文档
- 无法在服务端进行基于 `_source` 的 update 操作
- 无法仅依靠索引本身完成 [重建索引]({{< relref "/docs/operations/data-management/reindex.md" >}})（需要从外部数据源重拉）
- 无法使用 `_update_by_query` 批量更新

因此，**除非你有非常清晰的外部数据主源与恢复策略，否则不建议关闭 `_source`**。

## 小结

- `_source` 通常应保留——它是重建索引、调试与 update 操作的"最后后盾"
- `doc_values` 是聚合、排序、脚本操作的首选，默认已启用（`text` 字段除外）
- `stored fields` 在现代用法中已基本过时，仅在极少数特殊场景下使用
- 真正需要优化响应大小时，优先用 `_source_includes` / `_source_excludes` 做接口级投影
- 关闭 `_source` 会让 reindex、update 等操作无法在服务端完成，需谨慎评估

下一步可以继续阅读：

- [索引设置]({{< relref "/docs/operations/data-management/index-settings.md" >}})
- [别名（Aliases）]({{< relref "/docs/operations/data-management/aliases.md" >}})
- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})




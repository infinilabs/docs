---
title: "元数据参数（Meta）"
date: 0001-01-01
summary: "Meta 参数 #  meta 参数允许为字段附加自定义元数据。这些元数据不影响搜索或索引行为，仅用于记录字段的业务含义或管理信息。
相关指南 #    映射基础  示例 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;latency&#34;: { &#34;type&#34;: &#34;long&#34;, &#34;meta&#34;: { &#34;unit&#34;: &#34;ms&#34;, &#34;description&#34;: &#34;接口响应延迟&#34; } }, &#34;cpu_usage&#34;: { &#34;type&#34;: &#34;float&#34;, &#34;meta&#34;: { &#34;unit&#34;: &#34;percent&#34;, &#34;metric_type&#34;: &#34;gauge&#34; } } } } } 典型用途 #     场景 示例 meta 键值     标注计量单位 &quot;unit&quot;: &quot;ms&quot;, &quot;unit&quot;: &quot;bytes&quot;   标记指标类型 &quot;metric_type&quot;: &quot;counter&quot;, &quot;metric_type&quot;: &quot;gauge&quot;   记录字段用途 &quot;description&quot;: &quot;用户最后登录时间&quot;   标记数据来源 &quot;source&quot;: &quot;nginx_access_log&quot;   团队归属信息 &quot;owner&quot;: &quot;data-team&quot;    通过 API 查看 #  字段元数据会在 Get Mapping API 的响应中返回："
---


# Meta 参数

`meta` 参数允许为字段附加**自定义元数据**。这些元数据不影响搜索或索引行为，仅用于记录字段的业务含义或管理信息。

## 相关指南

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})

## 示例

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "latency": {
        "type": "long",
        "meta": {
          "unit": "ms",
          "description": "接口响应延迟"
        }
      },
      "cpu_usage": {
        "type": "float",
        "meta": {
          "unit": "percent",
          "metric_type": "gauge"
        }
      }
    }
  }
}
```

## 典型用途

| 场景 | 示例 `meta` 键值 |
|------|------------------|
| 标注计量单位 | `"unit": "ms"`, `"unit": "bytes"` |
| 标记指标类型 | `"metric_type": "counter"`, `"metric_type": "gauge"` |
| 记录字段用途 | `"description": "用户最后登录时间"` |
| 标记数据来源 | `"source": "nginx_access_log"` |
| 团队归属信息 | `"owner": "data-team"` |

## 通过 API 查看

字段元数据会在 Get Mapping API 的响应中返回：

```json
GET my-index/_mapping
```

响应中可以看到 `meta` 信息原样返回。

## 约束

| 约束 | 说明 |
|------|------|
| 键数量 | 最多 **5** 个键 |
| 键长度 | 最长 **20** 个字符 |
| 值类型 | 仅支持字符串 |
| 值长度 | 最长 **50** 个字符 |

## 注意事项

- `meta` 纯粹是**描述性的**，不会被索引、搜索或聚合
- 可以通过 Update Mapping API 修改已有字段的 `meta`，无需重建索引
- 适合配合 Kibana/INFINI Console 等可视化工具，为字段提供人类可读的附加信息


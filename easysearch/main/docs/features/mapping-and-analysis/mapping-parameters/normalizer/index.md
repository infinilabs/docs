---
title: "规范化器参数（Normalizer）"
date: 0001-01-01
summary: "Normalizer 参数 #  normalizer 参数用于 keyword 字段，在索引和查询之前对值进行标准化处理（如转换为小写）。与 analyzer 不同，normalizer 不会对字符串进行分词，只做字符级别的变换。
 完整指南 → 归一化与规范化器，包含概念介绍、自定义配置、兼容过滤器列表和最佳实践。
 参数选项 #     值 说明     null 不使用 normalizer，值按原样索引。默认值。   自定义名称 使用在索引 settings 中定义的 normalizer。    使用示例 #  keyword 字段默认区分大小写。使用内置的 lowercase normalizer 可忽略大小写：
&#34;status&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;normalizer&#34;: &#34;lowercase&#34; } 或自定义 normalizer：
PUT my-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;normalizer&#34;: { &#34;lowercase_normalizer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;filter&#34;: [&#34;lowercase&#34;] } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;status&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;normalizer&#34;: &#34;lowercase_normalizer&#34; } } } } 写入 &quot;OK&quot; 后，使用 &quot;ok&quot; 可以查到，因为两者在索引时都被标准化为 &quot;ok&quot;。"
---


# Normalizer 参数

`normalizer` 参数用于 `keyword` 字段，在索引和查询之前对值进行标准化处理（如转换为小写）。与 `analyzer` 不同，normalizer 不会对字符串进行分词，只做字符级别的变换。

> **完整指南** → [归一化与规范化器]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization" >}})，包含概念介绍、自定义配置、兼容过滤器列表和最佳实践。

## 参数选项

| 值 | 说明 |
|----|------|
| `null` | 不使用 normalizer，值按原样索引。**默认值**。 |
| `自定义名称` | 使用在索引 settings 中定义的 normalizer。 |

## 使用示例

`keyword` 字段默认区分大小写。使用内置的 `lowercase` normalizer 可忽略大小写：

```json
"status": {
  "type": "keyword",
  "normalizer": "lowercase"
}
```

或自定义 normalizer：

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "lowercase_normalizer": {
          "type": "custom",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "status": {
        "type": "keyword",
        "normalizer": "lowercase_normalizer"
      }
    }
  }
}
```

写入 `"OK"` 后，使用 `"ok"` 可以查到，因为两者在索引时都被标准化为 `"ok"`。

## 注意事项

- normalizer 同时影响**索引时**和**查询时**，确保两端一致
- 聚合和排序使用的也是标准化后的值
- 如果只需要查询时忽略大小写而保留原始值用于展示，可以使用多字段：一个带 normalizer，一个不带

## 相关参考

- [归一化与规范化器（概念指南）]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization" >}})
- [规范化器配置参考]({{< relref "/docs/features/mapping-and-analysis/text-analysis/normalizers/" >}})
- [Keyword 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/string-field-type/keyword" >}})


---
title: "简繁转换过滤器（STConvert Filter）"
date: 0001-01-01
summary: "简繁转换过滤器 #  stconvert 词元过滤器在简体中文和繁体中文之间进行转换，来自 analysis-stconvert 插件。
前提条件 #  需要安装 analysis-stconvert 插件：
bin/easysearch-plugin install analysis-stconvert 功能说明 #  此过滤器支持：
 简体 → 繁体（s2t） 繁体 → 简体（t2s）  可用于实现跨简繁的统一搜索。
使用示例 #  PUT my-stconvert-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;to_simplified&#34;: { &#34;type&#34;: &#34;stconvert&#34;, &#34;convert_type&#34;: &#34;t2s&#34; } }, &#34;analyzer&#34;: { &#34;unified_chinese&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;ik_smart&#34;, &#34;filter&#34;: [&#34;to_simplified&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;ik_smart&#34;, &#34;filter&#34;: [{&#34;type&#34;: &#34;stconvert&#34;, &#34;convert_type&#34;: &#34;t2s&#34;}], &#34;text&#34;: &#34;計算機程式設計&#34; } 响应：计算机 程式 设计"
---


# 简繁转换过滤器

`stconvert` 词元过滤器在简体中文和繁体中文之间进行转换，来自 analysis-stconvert 插件。

## 前提条件

需要安装 analysis-stconvert 插件：

```bash
bin/easysearch-plugin install analysis-stconvert
```

## 功能说明

此过滤器支持：

- 简体 → 繁体（`s2t`）
- 繁体 → 简体（`t2s`）

可用于实现跨简繁的统一搜索。

## 使用示例

```json
PUT my-stconvert-index
{
  "settings": {
    "analysis": {
      "filter": {
        "to_simplified": {
          "type": "stconvert",
          "convert_type": "t2s"
        }
      },
      "analyzer": {
        "unified_chinese": {
          "type": "custom",
          "tokenizer": "ik_smart",
          "filter": ["to_simplified"]
        }
      }
    }
  }
}
```

### 测试效果

```json
GET /_analyze
{
  "tokenizer": "ik_smart",
  "filter": [{"type": "stconvert", "convert_type": "t2s"}],
  "text": "計算機程式設計"
}
```

响应：`计算机` `程式` `设计`

## 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `convert_type` | `s2t` | 转换方向：`s2t`（简→繁）或 `t2s`（繁→简） |
| `delimiter` | `,` | 多音字拼音分隔符 |
| `keep_both` | `false` | 是否同时保留简体和繁体 |

## 相关链接

- [简繁转换分析器]({{< relref "../analyzers/stconvert-analyzer" >}})
- [简繁转换分词器]({{< relref "../tokenizers/stconvert" >}})



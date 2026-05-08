---
title: "ICU 排序过滤器（ICU Collation）"
date: 0001-01-01
summary: "ICU 排序过滤器 #  icu_collation_keyword 词元过滤器使用 ICU 排序规则将词元转换为排序键（collation key），实现语言感知的排序和范围查询。
前提条件 #  需要安装 analysis-icu 插件：
bin/easysearch-plugin install analysis-icu 功能说明 #  此过滤器将文本词元转换为 ICU CollationKey 的字节表示。转换后的词元可用于：
 语言感知排序：按特定语言的排序规则排列结果 范围查询：在 keyword 字段上执行符合语言习惯的范围过滤 重音/大小写不敏感匹配：通过调整排序强度控制匹配精度  使用示例 #  基本用法 — 德语排序 #  PUT my-german-sort { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;german_collation&#34;: { &#34;type&#34;: &#34;icu_collation_keyword&#34;, &#34;language&#34;: &#34;de&#34;, &#34;country&#34;: &#34;DE&#34; } }, &#34;analyzer&#34;: { &#34;german_sort&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;keyword&#34;, &#34;filter&#34;: [&#34;german_collation&#34;] } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;fields&#34;: { &#34;sort&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;analyzer&#34;: &#34;german_sort&#34; } } } } } } 忽略重音的匹配 #  PUT my-accent-insensitive { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;collation_primary&#34;: { &#34;type&#34;: &#34;icu_collation_keyword&#34;, &#34;language&#34;: &#34;en&#34;, &#34;strength&#34;: &#34;primary&#34; } } } } } 参数 #     参数 类型 说明     language string ICU 语言代码（如 de, fr, zh）   country string ICU 国家代码（如 DE, FR）   strength string 排序强度：primary（忽略重音+大小写）、secondary（区分重音）、tertiary（区分大小写）、quaternary、identical   decomposition string Unicode 分解模式：no、canonical   alternate string 空白/标点处理：shifted（忽略）、non-ignorable   caseLevel boolean 是否在 primary 强度下区分大小写   caseFirst string 大小写优先：lower、upper   numeric boolean 是否按数字值排序（true 使 &ldquo;2&rdquo; &lt; &ldquo;10&rdquo;）   rules string 自定义 ICU 排序规则字符串    相关链接 #    ICU 分析器  ICU Transform 过滤器  文本分析  "
---


# ICU 排序过滤器

`icu_collation_keyword` 词元过滤器使用 ICU 排序规则将词元转换为排序键（collation key），实现语言感知的排序和范围查询。

## 前提条件

需要安装 analysis-icu 插件：

```bash
bin/easysearch-plugin install analysis-icu
```

## 功能说明

此过滤器将文本词元转换为 ICU `CollationKey` 的字节表示。转换后的词元可用于：

- **语言感知排序**：按特定语言的排序规则排列结果
- **范围查询**：在 `keyword` 字段上执行符合语言习惯的范围过滤
- **重音/大小写不敏感匹配**：通过调整排序强度控制匹配精度

## 使用示例

### 基本用法 — 德语排序

```json
PUT my-german-sort
{
  "settings": {
    "analysis": {
      "filter": {
        "german_collation": {
          "type": "icu_collation_keyword",
          "language": "de",
          "country": "DE"
        }
      },
      "analyzer": {
        "german_sort": {
          "type": "custom",
          "tokenizer": "keyword",
          "filter": ["german_collation"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "sort": {
            "type": "keyword",
            "analyzer": "german_sort"
          }
        }
      }
    }
  }
}
```

### 忽略重音的匹配

```json
PUT my-accent-insensitive
{
  "settings": {
    "analysis": {
      "filter": {
        "collation_primary": {
          "type": "icu_collation_keyword",
          "language": "en",
          "strength": "primary"
        }
      }
    }
  }
}
```

## 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `language` | string | ICU 语言代码（如 `de`, `fr`, `zh`） |
| `country` | string | ICU 国家代码（如 `DE`, `FR`） |
| `strength` | string | 排序强度：`primary`（忽略重音+大小写）、`secondary`（区分重音）、`tertiary`（区分大小写）、`quaternary`、`identical` |
| `decomposition` | string | Unicode 分解模式：`no`、`canonical` |
| `alternate` | string | 空白/标点处理：`shifted`（忽略）、`non-ignorable` |
| `caseLevel` | boolean | 是否在 primary 强度下区分大小写 |
| `caseFirst` | string | 大小写优先：`lower`、`upper` |
| `numeric` | boolean | 是否按数字值排序（`true` 使 "2" < "10"） |
| `rules` | string | 自定义 ICU 排序规则字符串 |

## 相关链接

- [ICU 分析器]({{< relref "../analyzers/icu-analyzer" >}})
- [ICU Transform 过滤器]({{< relref "./icu-transform" >}})
- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})



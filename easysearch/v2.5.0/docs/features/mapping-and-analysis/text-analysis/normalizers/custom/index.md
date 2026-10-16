---
title: "自定义规范化器"
date: 0001-01-01
summary: "自定义规范化器 #  通过组合字符过滤器和词元过滤器，可以创建满足特定需求的自定义规范化器（normalizer）。
基本结构 #  PUT my-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;normalizer&#34;: { &#34;&lt;normalizer_name&gt;&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;char_filter&#34;: [], &#34;filter&#34;: [] } } } } } 参数 #     参数 类型 必选 说明     type string 是 固定为 custom   char_filter array 否 字符过滤器列表，在词元过滤器之前执行   filter array 否 词元过滤器列表     示例 #  大小写 + ASCII 折叠 #  最常用的组合——同时忽略大小写和变音符号："
---


# 自定义规范化器

通过组合字符过滤器和词元过滤器，可以创建满足特定需求的自定义规范化器（normalizer）。

## 基本结构

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "<normalizer_name>": {
          "type": "custom",
          "char_filter": [],
          "filter": []
        }
      }
    }
  }
}
```

### 参数

| 参数 | 类型 | 必选 | 说明 |
|------|------|------|------|
| `type` | string | 是 | 固定为 `custom` |
| `char_filter` | array | 否 | 字符过滤器列表，在词元过滤器之前执行 |
| `filter` | array | 否 | 词元过滤器列表 |

---

## 示例

### 大小写 + ASCII 折叠

最常用的组合——同时忽略大小写和变音符号：

```json
PUT sample-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "normalized_keyword": {
          "type": "custom",
          "char_filter": [],
          "filter": ["asciifolding", "lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "approach": {
        "type": "keyword",
        "normalizer": "normalized_keyword"
      }
    }
  }
}
```

索引一个文档：

```json
POST /sample-index/_doc/
{
  "approach": "naive"
}
```

以下查询可以匹配该文档：

```json
GET /sample-index/_search
{
  "query": {
    "term": {
      "approach": "Naïve"
    }
  }
}
```

因为 normalizer 将 `"Naïve"` 和 `"naive"` 都归一化为 `"naive"`。

验证归一化效果：

```json
GET /sample-index/_analyze
{
  "normalizer": "normalized_keyword",
  "text": "Naïve"
}
```

### 字符映射归一化

使用 `mapping` 字符过滤器替换特定字符，适合产品编码等场景：

```json
PUT product-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "dash_to_underscore": {
          "type": "mapping",
          "mappings": ["- => _"]
        }
      },
      "normalizer": {
        "product_code_normalizer": {
          "type": "custom",
          "char_filter": ["dash_to_underscore"],
          "filter": ["uppercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "product_code": {
        "type": "keyword",
        "normalizer": "product_code_normalizer"
      }
    }
  }
}
```

这样 `"abc-123"`、`"ABC-123"`、`"abc_123"` 都会被归一化为 `"ABC_123"`。

### CJK 全半角归一化

处理中日韩文本中的全角/半角字符差异：

```json
PUT cjk-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "cjk_normalizer": {
          "type": "custom",
          "filter": ["cjk_width", "lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "code": {
        "type": "keyword",
        "normalizer": "cjk_normalizer"
      }
    }
  }
}
```

全角字母 `"Ａｐｐｌｅ"` 和半角 `"Apple"` 都会归一化为 `"apple"`。

### 正则替换

使用 `pattern_replace` 字符过滤器去除空白或格式化字符：

```json
PUT clean-index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "strip_spaces": {
          "type": "pattern_replace",
          "pattern": "\\s+",
          "replacement": ""
        }
      },
      "normalizer": {
        "compact_normalizer": {
          "type": "custom",
          "char_filter": ["strip_spaces"],
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "serial": {
        "type": "keyword",
        "normalizer": "compact_normalizer"
      }
    }
  }
}
```

这样 `"AB CD 123"` 和 `"abcd123"` 都会归一化为 `"abcd123"`。

### 多语言归一化

对阿拉伯语关键字进行字符归一化：

```json
PUT arabic-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "arabic_normalizer": {
          "type": "custom",
          "filter": ["arabic_normalization", "lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "tag": {
        "type": "keyword",
        "normalizer": "arabic_normalizer"
      }
    }
  }
}
```

---

## 在映射中使用

定义后，在字段映射中通过 `normalizer` 参数引用：

```json
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      }
    }
  }
}
```

### 多字段配合

如果需要同时保留精确值和归一化值，使用多字段：

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "folding_normalizer": {
          "type": "custom",
          "filter": ["asciifolding", "lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "brand": {
        "type": "keyword",
        "fields": {
          "normalized": {
            "type": "keyword",
            "normalizer": "folding_normalizer"
          }
        }
      }
    }
  }
}
```

- `brand` — 精确匹配，区分大小写
- `brand.normalized` — 归一化匹配，忽略大小写和变音符号

---

## 可用的过滤器

自定义 normalizer 只能使用字符级操作的过滤器，完整列表请参阅[规范化器概述]({{< relref "../normalizers/" >}})。

## 注意事项

1. **过滤器顺序有影响**：`["asciifolding", "lowercase"]` 先移除变音符号再转小写，通常这个顺序最佳
2. **char_filter 先于 filter 执行**：字符过滤器在词元过滤器之前处理
3. **不支持分词器**：normalizer 永远不会把一个值拆成多个词元
4. **索引内定义**：每个 normalizer 定义在具体索引的 settings 中，不能跨索引共享


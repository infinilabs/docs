---
title: "存储脚本"
date: 0001-01-01
summary: "存储脚本 #  存储脚本（Stored Scripts）允许您将常用的脚本预先保存在集群中，之后通过 ID 引用执行。使用存储脚本可以减少请求体大小、避免重复编译、并集中管理脚本逻辑。
创建存储脚本 #  使用 POST _scripts/&lt;script_id&gt; 创建或更新存储脚本：
POST _scripts/calculate_discount { &#34;script&#34;: { &#34;lang&#34;: &#34;painless&#34;, &#34;source&#34;: &#34;doc[&#39;price&#39;].value * (1 - params.discount_rate)&#34; } } 指定脚本上下文 #  可以在创建时指定脚本的使用上下文，Easysearch 会在存储时进行上下文相关的语法校验：
POST _scripts/my_score_script/score { &#34;script&#34;: { &#34;lang&#34;: &#34;painless&#34;, &#34;source&#34;: &#34;_score * doc[&#39;boost&#39;].value&#34; } } 上下文名称放在 URL 路径的第二段（/score），可选值包括：score、filter、update、ingest 等。
获取存储脚本 #  GET _scripts/calculate_discount 响应：
{ &#34;_id&#34;: &#34;calculate_discount&#34;, &#34;found&#34;: true, &#34;script&#34;: { &#34;lang&#34;: &#34;painless&#34;, &#34;source&#34;: &#34;doc[&#39;price&#39;].value * (1 - params."
---


# 存储脚本

存储脚本（Stored Scripts）允许您将常用的脚本预先保存在集群中，之后通过 ID 引用执行。使用存储脚本可以减少请求体大小、避免重复编译、并集中管理脚本逻辑。

## 创建存储脚本

使用 `POST _scripts/<script_id>` 创建或更新存储脚本：

```json
POST _scripts/calculate_discount
{
  "script": {
    "lang": "painless",
    "source": "doc['price'].value * (1 - params.discount_rate)"
  }
}
```

### 指定脚本上下文

可以在创建时指定脚本的使用上下文，Easysearch 会在存储时进行上下文相关的语法校验：

```json
POST _scripts/my_score_script/score
{
  "script": {
    "lang": "painless",
    "source": "_score * doc['boost'].value"
  }
}
```

上下文名称放在 URL 路径的第二段（`/score`），可选值包括：`score`、`filter`、`update`、`ingest` 等。

## 获取存储脚本

```json
GET _scripts/calculate_discount
```

响应：

```json
{
  "_id": "calculate_discount",
  "found": true,
  "script": {
    "lang": "painless",
    "source": "doc['price'].value * (1 - params.discount_rate)"
  }
}
```

## 使用存储脚本

在查询、更新、聚合等操作中通过 `id` 引用存储脚本：

### 在搜索中使用

```json
GET products/_search
{
  "script_fields": {
    "final_price": {
      "script": {
        "id": "calculate_discount",
        "params": {
          "discount_rate": 0.15
        }
      }
    }
  }
}
```

### 在 Script Score 中使用

```json
GET products/_search
{
  "query": {
    "script_score": {
      "query": { "match": { "title": "手机" } },
      "script": {
        "id": "my_score_script"
      }
    }
  }
}
```

### 在 Update 中使用

```json
POST _scripts/increment_counter
{
  "script": {
    "lang": "painless",
    "source": "ctx._source[params.field] += params.amount"
  }
}
```

```json
POST my_index/_update/1
{
  "script": {
    "id": "increment_counter",
    "params": {
      "field": "view_count",
      "amount": 1
    }
  }
}
```

### 在 Search Template 中使用

存储脚本也可以作为 Mustache 搜索模板使用：

```json
POST _scripts/my_search_template
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "match": {
          "{{field}}": "{{query}}"
        }
      },
      "from": "{{from}}",
      "size": "{{size}}"
    }
  }
}
```

```json
GET my_index/_search/template
{
  "id": "my_search_template",
  "params": {
    "field": "title",
    "query": "Easysearch",
    "from": 0,
    "size": 10
  }
}
```

## 删除存储脚本

```json
DELETE _scripts/calculate_discount
```

## 查看可用脚本语言

```json
GET _script_language
```

## 查看可用脚本上下文

```json
GET _script_context
```

## API 参考

| 操作 | API |
|------|-----|
| 创建/更新脚本 | `POST _scripts/<id>` 或 `PUT _scripts/<id>` |
| 创建并校验上下文 | `POST _scripts/<id>/<context>` |
| 获取脚本 | `GET _scripts/<id>` |
| 删除脚本 | `DELETE _scripts/<id>` |
| 查看脚本语言 | `GET _script_language` |
| 查看脚本上下文 | `GET _script_context` |

## 最佳实践

1. **命名规范**：使用有意义的名称，如 `calculate_discount`、`normalize_score`，便于管理
2. **参数化设计**：将可变部分通过 `params` 传入，使同一脚本适用于不同场景
3. **版本管理**：存储脚本没有内置版本控制，建议在外部（如 Git）管理脚本源码，通过 CI/CD 部署
4. **上下文校验**：创建时指定上下文（如 `/score`），可以提前发现语法错误
5. **安全控制**：通过 `script.allowed_types` 设置可以限制只允许使用 `stored` 类型的脚本，禁止内联脚本


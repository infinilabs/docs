---
title: "Lucene 表达式"
date: 0001-01-01
summary: "Lucene 表达式 脚本语言 #  Lucene Expressions 是一种轻量级的数学表达式脚本语言，编译为 Java 字节码，性能极高。适用于简单的数值计算场景，如自定义排序和评分。
特点 #     特性 说明     性能 极高，直接编译为字节码，无解释器开销   安全 完全沙盒，只能进行数学运算   功能范围 仅支持数值运算和比较   数据访问 只能访问 doc values（doc['field']）   返回类型 始终返回 double    语法 #  基本运算 #  doc[&#39;price&#39;].value * 0.9 doc[&#39;score&#39;].value + doc[&#39;bonus&#39;].value (doc[&#39;a&#39;].value + doc[&#39;b&#39;].value) / 2 数学函数 #     函数 说明 示例     abs(x) 绝对值 abs(doc['delta']."
---


# Lucene 表达式 脚本语言

Lucene Expressions 是一种轻量级的数学表达式脚本语言，编译为 Java 字节码，性能极高。适用于简单的数值计算场景，如自定义排序和评分。

## 特点

| 特性 | 说明 |
|------|------|
| 性能 | 极高，直接编译为字节码，无解释器开销 |
| 安全 | 完全沙盒，只能进行数学运算 |
| 功能范围 | 仅支持数值运算和比较 |
| 数据访问 | 只能访问 doc values（`doc['field']`） |
| 返回类型 | 始终返回 `double` |

## 语法

### 基本运算

```
doc['price'].value * 0.9
doc['score'].value + doc['bonus'].value
(doc['a'].value + doc['b'].value) / 2
```

### 数学函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `abs(x)` | 绝对值 | `abs(doc['delta'].value)` |
| `sqrt(x)` | 平方根 | `sqrt(doc['area'].value)` |
| `pow(x, y)` | x 的 y 次方 | `pow(doc['base'].value, 2)` |
| `exp(x)` | e 的 x 次方 | `exp(-0.01 * doc['age'].value)` |
| `ln(x)` | 自然对数 | `ln(doc['count'].value + 1)` |
| `log10(x)` | 以 10 为底的对数 | `log10(doc['views'].value + 1)` |
| `log2(x)` | 以 2 为底的对数 | `log2(doc['size'].value)` |
| `ceil(x)` | 向上取整 | `ceil(doc['score'].value)` |
| `floor(x)` | 向下取整 | `floor(doc['score'].value)` |
| `max(a, b)` | 较大值 | `max(doc['a'].value, doc['b'].value)` |
| `min(a, b)` | 较小值 | `min(doc['a'].value, 100)` |
| `sin(x)`, `cos(x)`, `tan(x)` | 三角函数 | `sin(doc['angle'].value)` |
| `asin(x)`, `acos(x)`, `atan(x)` | 反三角函数 | `acos(doc['cos_val'].value)` |
| `atan2(y, x)` | 二参数反正切 | `atan2(doc['y'].value, doc['x'].value)` |
| `haversin(lat1, lon1, lat2, lon2)` | 球面距离（千米） | `haversin(doc['lat'].value, doc['lon'].value, 39.9, 116.4)` |

### 比较与条件

Expressions 不支持 `if-else`，但支持三元运算和布尔转换：

```
// 布尔值会转为 double（true=1.0, false=0.0）
doc['in_stock'].value ? doc['price'].value : 0

// 比较运算
doc['age'].value > 18 ? 1 : 0
```

## 使用示例

### 自定义排序

```json
GET products/_search
{
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "lang": "expression",
          "source": "doc['sales'].value * 0.7 + doc['rating'].value * 0.3"
        },
        "order": "desc"
      }
    }
  ]
}
```

### function_score 中使用

```json
GET articles/_search
{
  "query": {
    "function_score": {
      "query": { "match": { "title": "搜索" } },
      "script_score": {
        "script": {
          "lang": "expression",
          "source": "_score * ln(doc['views'].value + 1)"
        }
      }
    }
  }
}
```

### 地理距离评分

```json
GET shops/_search
{
  "query": {
    "function_score": {
      "query": { "match_all": {} },
      "script_score": {
        "script": {
          "lang": "expression",
          "source": "1.0 / (1 + haversin(doc['location'].lat, doc['location'].lon, 39.9042, 116.4074))"
        }
      }
    }
  }
}
```

## Expressions vs Painless

| 对比项 | Expressions | Painless |
|--------|-------------|----------|
| 性能 | ⚡ 极高 | ⚡ 高 |
| 功能范围 | 仅数学运算 | 通用（字符串、数组、条件…） |
| 数据访问 | 仅 doc values | doc values + _source + ctx |
| 控制流 | 仅三元运算 | if/else、for、while |
| 字符串操作 | ❌ | ✅ |
| 适用场景 | 简单评分/排序公式 | 复杂业务逻辑 |

> **建议**：如果只需要数值计算（如评分公式、距离计算），优先使用 Expressions 以获得最佳性能。对于需要条件逻辑、字符串处理、数组操作等复杂场景，使用 Painless。

## 限制

- 只能返回 `double` 类型的值
- 只能通过 `doc['field']` 访问 doc values 中的数值字段
- 不支持字符串操作、数组操作
- 不支持 `if-else`（只能用三元运算符）
- 不支持变量声明、循环等语句
- 不能用于 Update、Ingest 等修改文档的场景


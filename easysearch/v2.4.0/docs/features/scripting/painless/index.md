---
title: "Painless 脚本语言"
date: 0001-01-01
summary: "Painless 脚本语言 #  Painless 是 Easysearch 内置的高性能脚本语言，也是默认脚本语言。它专为搜索引擎场景设计，具备以下特点：
 高性能：编译为 Java 字节码，通过 JIT 直接在 JVM 上运行 安全沙盒：只允许访问白名单内的 Java API，无法执行文件、网络等危险操作 Java 风格语法：对 Java 开发者友好，学习成本低 专用优化：内置 doc、_source、ctx 等快捷访问方式  基本语法 #  变量与类型 #  Painless 是强类型语言，支持类型推断：
// 显式类型声明 int count = 10; double price = 99.5; String name = &#34;Easysearch&#34;; boolean active = true; // 类型推断（使用 def） def value = doc[&#39;price&#39;].value; def list = [1, 2, 3]; def map = [&#39;key&#39;: &#39;value&#39;, &#39;count&#39;: 42]; 支持的基本类型："
---


# Painless 脚本语言

Painless 是 Easysearch 内置的高性能脚本语言，也是默认脚本语言。它专为搜索引擎场景设计，具备以下特点：

- **高性能**：编译为 Java 字节码，通过 JIT 直接在 JVM 上运行
- **安全沙盒**：只允许访问白名单内的 Java API，无法执行文件、网络等危险操作
- **Java 风格语法**：对 Java 开发者友好，学习成本低
- **专用优化**：内置 `doc`、`_source`、`ctx` 等快捷访问方式

## 基本语法

### 变量与类型

Painless 是强类型语言，支持类型推断：

```painless
// 显式类型声明
int count = 10;
double price = 99.5;
String name = "Easysearch";
boolean active = true;

// 类型推断（使用 def）
def value = doc['price'].value;
def list = [1, 2, 3];
def map = ['key': 'value', 'count': 42];
```

支持的基本类型：

| 类型 | 说明 | 示例 |
|------|------|------|
| `byte`, `short`, `int`, `long` | 整数类型 | `42`, `100L` |
| `float`, `double` | 浮点类型 | `3.14`, `2.0f` |
| `boolean` | 布尔类型 | `true`, `false` |
| `char` | 字符类型 | `'A'` |
| `String` | 字符串 | `"hello"` |
| `def` | 动态类型（运行时确定） | `def x = 1` |

### 运算符

```painless
// 算术运算
int sum = a + b;
double avg = total / count;
int remainder = x % 2;

// 比较运算
boolean eq = (a == b);      // 值相等
boolean ref = (a === b);    // 引用相等
boolean gt = (a > b);

// 逻辑运算
boolean result = (a > 0) && (b < 100);
boolean either = (a > 0) || (b > 0);
boolean not = !(a > 0);

// 字符串连接
String full = first + " " + last;

// 三元运算符
def status = (score > 80) ? "pass" : "fail";

// 空值安全运算符（Elvis）
def val = params.value ?: 0;

// 空安全访问
def len = params.name?.length();
```

### 控制流

```painless
// if-else
if (doc['status'].value == 'active') {
  return 1;
} else if (doc['status'].value == 'pending') {
  return 0.5;
} else {
  return 0;
}

// for 循环
int total = 0;
for (int i = 0; i < 10; i++) {
  total += i;
}

// for-each 循环
for (def item : params.items) {
  total += item;
}

// while 循环
int i = 0;
while (i < doc['tags'].length) {
  if (doc['tags'][i] == 'important') {
    return true;
  }
  i++;
}
```

### 集合操作

```painless
// List（列表）
def list = [1, 2, 3, 4, 5];
list.add(6);
int size = list.size();
def first = list.get(0);
list.remove(0);

// Map（字典）
def map = ['name': 'test', 'score': 95];
map.put('grade', 'A');
def name = map.get('name');
boolean has = map.containsKey('score');

// 遍历 Map
for (def entry : map.entrySet()) {
  String key = entry.getKey();
  def val = entry.getValue();
}

// Stream 风格操作
def filtered = list.stream()
  .filter(x -> x > 2)
  .map(x -> x * 2)
  .collect(Collectors.toList());
```

### 正则表达式

```painless
// 匹配检查
if (doc['email'].value =~ /.*@example\.com/) {
  return true;
}

// 查找与提取
def matcher = /(\d{4})-(\d{2})-(\d{2})/.matcher(doc['date'].value);
if (matcher.find()) {
  def year = matcher.group(1);
}

// 替换
String cleaned = doc['text'].value.replaceAll(/\s+/, ' ');
```

## 访问文档数据

Painless 提供三种方式访问文档数据，适用于不同场景：

### 1. `doc['field']` — Doc Values 访问（推荐）

从列式存储的 doc values 中读取，**性能最好**：

```painless
// 读取单值字段
def price = doc['price'].value;
def name = doc['name.keyword'].value;

// 读取多值字段
def tags = doc['tags'];
int tagCount = tags.size();
def firstTag = tags[0];

// 检查字段是否有值
if (doc['field'].size() > 0) {
  return doc['field'].value;
}

// 日期字段
ZonedDateTime date = doc['timestamp'].value;
int year = date.getYear();
int month = date.getMonthValue();
```

> **注意**：`doc` 方式需要字段启用了 doc values（大多数字段默认启用，`text` 字段除外）。`text` 字段需要开启 `fielddata: true` 才能通过 `doc` 访问。

### 2. `params._source` — 源文档访问

从存储的 `_source` JSON 中读取，可以访问任意字段但**性能较慢**：

```painless
// 访问嵌套字段
def address = params._source.address.city;

// 访问 text 字段原始值
def description = params._source.description;

// 访问数组
def firstItem = params._source.items[0].name;
```

> **注意**：`params._source` 不是在所有上下文中都可用。在 `score` 和 `filter` 上下文中，推荐使用 `doc` 方式。

### 3. `ctx` — 上下文变量

在 Update 和 Ingest 场景中使用：

```painless
// Update API 中
ctx._source.counter += 1;
ctx._source.tags.add('new_tag');
ctx._source.last_updated = params.now;

// 条件删除
if (ctx._source.status == 'expired') {
  ctx.op = 'delete';
}

// Ingest 处理器中
ctx.field_name = 'new_value';
ctx.timestamp = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
```

### 访问方式对比

| 方式 | 性能 | 可用上下文 | 适用场景 |
|------|------|-----------|----------|
| `doc['field']` | ⚡ 最快 | Score、Filter、Agg、Sort | 查询时的字段访问 |
| `params._source` | 🐢 较慢 | 部分查询上下文 | 访问嵌套结构或 text 字段 |
| `ctx._source` | ⚡ 快 | Update、Ingest | 文档更新和摄入处理 |
| `params` | ⚡ 快 | 所有上下文 | 访问用户传入的参数 |

## 脚本上下文

Painless 脚本在不同的上下文中运行，每个上下文有不同的可用变量和返回值要求：

### Score 上下文（评分）

用于 `script_score` 查询，**必须返回 double**：

```json
GET my_index/_search
{
  "query": {
    "script_score": {
      "query": { "match_all": {} },
      "script": {
        "source": "doc['popularity'].value * Math.log(2 + doc['votes'].value)"
      }
    }
  }
}
```

可用变量：`doc`、`params`、`_score`（原始查询评分）

### Filter 上下文（过滤）

用于 `script` 查询，**必须返回 boolean**：

```json
GET my_index/_search
{
  "query": {
    "script": {
      "script": {
        "source": "doc['age'].value > params.min_age && doc['age'].value < params.max_age",
        "params": {
          "min_age": 18,
          "max_age": 65
        }
      }
    }
  }
}
```

可用变量：`doc`、`params`

### Update 上下文

用于 Update API，通过 `ctx._source` 修改文档：

```json
POST my_index/_update/1
{
  "script": {
    "source": "ctx._source.counter += params.increment",
    "params": {
      "increment": 5
    }
  }
}
```

可用变量：`ctx`（含 `_source`、`_id`、`_version`、`_routing`、`op`）、`params`

### Ingest 上下文

用于摄入管道的 Script Processor：

```json
PUT _ingest/pipeline/my_pipeline
{
  "processors": [
    {
      "script": {
        "source": "ctx.full_name = ctx.first_name + ' ' + ctx.last_name"
      }
    }
  ]
}
```

可用变量：`ctx`（文档字段）、`params`

### Aggregation 上下文

用于 Scripted Metric Aggregation：

```json
GET my_index/_search
{
  "aggs": {
    "profit": {
      "scripted_metric": {
        "init_script": "state.transactions = []",
        "map_script": "state.transactions.add(doc['profit'].value)",
        "combine_script": "double total = 0; for (t in state.transactions) { total += t; } return total",
        "reduce_script": "double total = 0; for (s in states) { total += s; } return total"
      }
    }
  }
}
```

### Sort 上下文（排序）

用于自定义排序，返回排序值：

```json
GET my_index/_search
{
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "source": "doc['priority'].value * 100 + doc['score'].value"
        },
        "order": "desc"
      }
    }
  ]
}
```

可用变量：`doc`、`params`

## 常用内置 API

### 数学函数

```painless
Math.abs(-5)           // 5
Math.max(a, b)         // 较大值
Math.min(a, b)         // 较小值
Math.pow(2, 10)        // 1024.0
Math.sqrt(144)         // 12.0
Math.log(100)          // 4.605...
Math.log10(100)        // 2.0
Math.ceil(3.2)         // 4.0
Math.floor(3.8)        // 3.0
Math.round(3.5)        // 4
Math.random()          // 0.0-1.0 随机数
```

### 字符串方法

```painless
String s = "Hello World";
s.length()                  // 11
s.substring(0, 5)           // "Hello"
s.toLowerCase()             // "hello world"
s.toUpperCase()             // "HELLO WORLD"
s.trim()                    // 去除首尾空白
s.contains("World")         // true
s.startsWith("Hello")       // true
s.endsWith("World")         // true
s.indexOf("World")          // 6
s.replace("World", "ES")    // "Hello ES"
s.split(" ")                // ["Hello", "World"]
```

### 日期与时间

```painless
// 从 doc values 获取日期
ZonedDateTime dt = doc['timestamp'].value;
dt.getYear()              // 2025
dt.getMonthValue()        // 6
dt.getDayOfMonth()        // 15
dt.getHour()              // 14
dt.getMinute()            // 30

// 日期差值（毫秒）
long now = System.currentTimeMillis();
long docTime = doc['timestamp'].value.toInstant().toEpochMilli();
long diffDays = (now - docTime) / (1000 * 60 * 60 * 24);

// 格式化日期
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String dateStr = dt.format(formatter);
```

## 常见使用示例

### 条件赋分

```json
GET products/_search
{
  "query": {
    "script_score": {
      "query": { "match": { "title": "手机" } },
      "script": {
        "source": """
          double score = _score;
          if (doc['in_stock'].value) {
            score *= 1.5;
          }
          if (doc.containsKey('promotion') && doc['promotion'].size() > 0) {
            score *= 2.0;
          }
          return score;
        """
      }
    }
  }
}
```

### 批量字段更新

```json
POST my_index/_update_by_query
{
  "script": {
    "source": """
      ctx._source.full_name = ctx._source.first_name + ' ' + ctx._source.last_name;
      if (ctx._source.age >= 18) {
        ctx._source.category = 'adult';
      } else {
        ctx._source.category = 'minor';
      }
    """
  },
  "query": {
    "bool": {
      "must_not": {
        "exists": { "field": "full_name" }
      }
    }
  }
}
```

### 数组操作

```json
POST my_index/_update/1
{
  "script": {
    "source": """
      if (!ctx._source.tags.contains(params.new_tag)) {
        ctx._source.tags.add(params.new_tag);
      }
    """,
    "params": {
      "new_tag": "featured"
    }
  }
}
```

### 按天衰减评分

```json
GET articles/_search
{
  "query": {
    "script_score": {
      "query": { "match": { "content": "搜索引擎" } },
      "script": {
        "source": """
          long now = System.currentTimeMillis();
          long published = doc['publish_date'].value.toInstant().toEpochMilli();
          long daysDiff = (now - published) / (1000L * 60 * 60 * 24);
          double decay = Math.exp(-0.01 * daysDiff);
          return _score * decay;
        """
      }
    }
  }
}
```

## 调试与排查

### 使用 Painless Execute API 调试

```json
POST _scripts/painless/_execute
{
  "script": {
    "source": """
      int total = 0;
      for (int i = 0; i < params.values.size(); i++) {
        total += params.values[i];
      }
      return total;
    """,
    "params": {
      "values": [10, 20, 30]
    }
  }
}
```

### 使用 Explain API 调试评分

```json
GET my_index/_explain/1
{
  "query": {
    "script_score": {
      "query": { "match_all": {} },
      "script": {
        "source": "doc['score'].value * 2"
      }
    }
  }
}
```

### 常见错误与解决

| 错误信息 | 原因 | 解决方法 |
|---------|------|---------|
| `compile error` | 语法错误 | 检查语法，使用 Painless Execute API 调试 |
| `null_pointer_exception` | 字段值为空 | 使用 `doc['field'].size() > 0` 检查 |
| `class_cast_exception` | 类型不匹配 | 使用显式类型转换或 `def` |
| `script_exception: too many compilations` | 编译频率过高 | 使用 `params` 参数化，避免动态拼接脚本 |

## 性能最佳实践

1. **使用 `params` 参数化**：将变量值通过 `params` 传入，使脚本可被缓存复用
2. **优先使用 `doc` 访问**：`doc['field'].value` 比 `params._source.field` 快很多
3. **避免在热路径使用脚本**：高频查询优先考虑非脚本方案（如 `function_score` 内置函数）
4. **使用存储脚本**：反复使用的脚本存储后可避免重复编译
5. **控制脚本复杂度**：避免在脚本中做大量循环或复杂计算
6. **使用强类型**：明确声明类型（`int`、`double`）比 `def` 性能稍好


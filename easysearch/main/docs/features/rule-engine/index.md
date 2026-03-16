---
title: "规则引擎"
date: 0001-01-01
description: "高性能实时规则匹配引擎，支持在数据入库瞬间完成关键词检测、自动打标与内容清洗。"
summary: "Rules 规则引擎 #  Rules 插件是 EasySearch 的规则匹配引擎，可在数据写入时自动匹配规则库，为文档添加标签字段，实现高效的内容分类、审核和标注。
功能特性 #   ✅ 实时匹配：数据写入时自动匹配规则，无需后处理 ✅ 高性能：基于优化的 C++ 规则引擎，支持复杂规则和大规则库 ✅ 灵活配置：支持自定义字段、正则表达式、数值范围匹配 ✅ 多规则匹配：一条文档可匹配多个规则，所有标签保留到数组中 ✅ 集群广播：多节点环境自动编译规则到所有 Ingest 节点   基本概念 #  规则库 (Repository) #  规则库是规则的集合，每个规则库有唯一的 repo_id，存储在 .match_rules 索引中。
规则 (Rule) #  规则由表达式和描述组成：
expression\t#offset#description  expression: 匹配表达式（支持 AND/OR/NOT、正则、范围等） offset: 规则序号（从 0 开始，系统自动生成，用于区分不同规则） description: 规则描述（匹配时作为标签返回）  重要说明：
 offset 是导入到规则索引后系统自动生成的，本身没有业务含义 通过 _import API 导入规则时不需要手动指定 offset 导入时只需提供 expression 和 description，系统会自动分配 offset  标签 (Tag) #  文档匹配规则后，规则的 #offset#description 会添加到文档的 tags 字段。"
---


# Rules 规则引擎

Rules 插件是 EasySearch 的规则匹配引擎，可在数据写入时自动匹配规则库，为文档添加标签字段，实现高效的内容分类、审核和标注。

## 功能特性

- ✅ **实时匹配**：数据写入时自动匹配规则，无需后处理
- ✅ **高性能**：基于优化的 C++ 规则引擎，支持复杂规则和大规则库
- ✅ **灵活配置**：支持自定义字段、正则表达式、数值范围匹配
- ✅ **多规则匹配**：一条文档可匹配多个规则，所有标签保留到数组中
- ✅ **集群广播**：多节点环境自动编译规则到所有 Ingest 节点

---

## 基本概念

### 规则库 (Repository)

规则库是规则的集合，每个规则库有唯一的 `repo_id`，存储在 `.match_rules` 索引中。

### 规则 (Rule)

规则由**表达式**和**描述**组成：

```
expression\t#offset#description
```

- **expression**: 匹配表达式（支持 AND/OR/NOT、正则、范围等）
- **offset**: 规则序号（从 0 开始，系统自动生成，用于区分不同规则）
- **description**: 规则描述（匹配时作为标签返回）

**重要说明**：
- `offset` 是导入到规则索引后系统自动生成的，本身没有业务含义
- 通过 `_import` API 导入规则时**不需要**手动指定 offset
- 导入时只需提供 `expression` 和 `description`，系统会自动分配 offset

### 标签 (Tag)

文档匹配规则后，规则的 `#offset#description` 会添加到文档的 `tags` 字段。

---

## 环境要求

### 节点配置

Rules 插件依赖 native 库（C++ 编译的规则引擎），需要在 `config/easysearch.yml` 中禁用系统调用过滤器：

```yaml
bootstrap.system_call_filter: false
```

**为什么需要这个配置？**

- Rules 插件需要调用 JNI 和 native 库（libruledb-r.so）
- 规则编译过程会执行系统调用（如 fork、exec 等）
- EasySearch 默认启用的 seccomp 过滤器会阻止这些操作

**注意**：禁用此过滤器会降低一定的安全性，建议只在运行 rules 插件的节点上禁用。

---

## 快速开始

### 步骤 1: 导入规则

使用 `POST /_match_rules/{repo_id}/_import` 导入规则到索引：

```json
POST /_match_rules/security_v1/_import
{
  "name": "安全规则库",
  "description": "用于安全事件分类的规则库",
  "tags": ["security", "content-filter"],
  "rules": [
    {
      "expression": "枪 or 手枪 or 步枪 or 猎枪",
      "description": "涉枪武器关键词"
    },
    {
      "expression": "AK47 or AK-47 or M16 or M4A1",
      "description": "枪支型号"
    },
    {
      "expression": "海洛因 or 冰毒 or 摇头丸 or K粉",
      "description": "毒品名称"
    }
  ]
}
```

**响应**：
```json
{
  "success": true,
  "message": "Successfully imported 3 rules to repo_id 'security_v1' (version: 1)",
  "repo_id": "security_v1",
  "version": "1",
  "rule_count": 3
}
```

### 步骤 2: 编译规则库

使用 `POST /_match_rules/{repo_id}/_compile` 编译规则为二进制库：

```json
POST /_match_rules/security_v1/_compile
{
  "fields": ["title", "content", "range", "price"],
  "composite": ["(title,content)", "(price,range)"],
  "quiet": true
}
```

**参数说明**：

| 参数 | 类型 | 必需 | 默认值 | 说明 |
|------|------|------|--------|------|
| `fields` | array | ❌ | [] | 编译时声明的字段列表（用于字段限定与范围匹配等） |
| `composite` | array | ❌ | [] | 联合索引定义（用于多字段组合匹配，提升性能） |
| `quiet` | boolean | ❌ | true | 是否静默模式（不输出详细日志） |

**联合索引说明**：

`composite` 参数用于定义多字段组合索引，提升多字段 AND 条件的匹配性能：

```json
{
  "fields": ["gender", "age", "income"],
  "composite": [
    "(gender,age,income)",           // 基本格式
    "s{:}(name,city,province)"       // 带分隔符格式
  ]
}
```

**格式说明**：
- `(field1,field2,field3)`: 基本格式，多个字段组合索引
- `s{sep}(field1,field2)`: 带分隔符格式，`sep` 为自定义分隔符（如 `:`）

**性能优势**：
- 单字段索引需要分别扫描 + 内存交集
- 联合索引一次扫描组合索引，性能提升 3-10 倍

**推荐使用场景**：
- 规则中频繁出现 `field1 and field2 and field3` 的组合
- 字段基数较小，组合后索引膨胀可控
- 对匹配性能要求高的场景

**响应**：
```json
{
  "success": true,
  "message": "Successfully compiled 3 rules",
  "result": {
    "duration_ms": 1250,
    "total_rules": 3,
    "indexed_rules": 0
  }
}
```

### 联合索引配置指南

当规则中频繁出现多字段 AND 组合（如 `author(...) and source(...)`）时，建议配置 `composite`。

**步骤 1：确定规则中的高频字段组合**

规则示例：
```text
author(张三) and source(新华网)
author(张三) and title(告警)
```

**步骤 2：编译时声明 `fields` 与 `composite`**

```json
POST /_match_rules/news_v1/_compile
{
  "fields": ["author", "source", "title"],
  "composite": [
    "(author,source)",
    "(author,title)",
    "s{:}(author,source,title)"
  ],
  "quiet": true
}
```

说明：
- `fields`：声明编译器需要处理的字段
- `composite`：声明联合索引定义，编译时会作为 `-i` 参数传给编译器
- `s{sep}(...)`：可选分隔符格式，`sep` 例如 `:`

**步骤 3：验证配置已生效**

```json
GET .match_rules/_doc/news_v1
```

预期 `_source` 包含：
```json
{
  "fields": ["author", "source", "title"],
  "composite": ["(author,source)", "(author,title)", "s{:}(author,source,title)"]
}
```

**步骤 4：在 Pipeline 中按需设置字段白名单**

```json
PUT _ingest/pipeline/news-check
{
  "processors": [
    {
      "check_match_rules": {
        "id": "news_v1",
        "fields": ["author", "source", "title"]
      }
    }
  ]
}
```

**常见错误**
- `composite` 格式错误（如 `author,source` 缺少括号）会导致编译失败
- 只配置 `composite` 不配置相关 `fields`，通常会降低可维护性（建议同时声明）
- 未重编译直接测试，新配置不会生效

### 步骤 3: 创建 Ingest Pipeline

```json
PUT _ingest/pipeline/security-check
{
  "description": "使用规则引擎对安全事件分类",
  "processors": [
    {
      "check_match_rules": {
        "id": "security_v1"
      }
    }
  ]
}
```

**使用自定义目标字段**：

```json
PUT _ingest/pipeline/security-check-custom
{
  "description": "使用规则引擎对安全事件分类，结果写入 security_labels 字段",
  "processors": [
    {
      "check_match_rules": {
        "id": "security_v1",
        "target_field": "security_labels"
      }
    }
  ]
}
```

**配置参数**：

| 参数 | 类型 | 必需 | 默认值 | 说明                                     |
|------|------|------|--------|----------------------------------------|
| `id` | string | ✅ | - | 规则库 ID（对应 repo_id）                     |
| `target_field` | string | ❌ | "tags" | 匹配结果写入的目标字段名                           |
| `ignore_missing` | boolean | ❌ | false | 是否忽略缺失字段错误                             |
| `regex_start_at_word` | boolean | ❌ | true | 正则是否从单词边界开始                            |
| `fields` | array | ❌ | [] | 文档字段白名单：指定后仅这些字段按原字段名参与匹配，其余字段内容汇总到 `default_match_field` |
| `default_match_field` | string | ❌ | "content" | 当配置了 `fields` 时，未包含字段会汇总到该字段名参与匹配 |

### 步骤 4: 写入文档测试

```json
POST security-events/_doc?pipeline=security-check
{
  "title": "枪支交易案",
  "content": "网上出售手枪，价格5000元",
  "timestamp": "2025-12-31T10:00:00Z"
}
```

**查询结果**：
```json
{
  "_source": {
    "title": "枪支交易案",
    "content": "网上出售手枪，价格5000元",
    "timestamp": "2025-12-31T10:00:00Z",
    "tags": ["#0#涉枪武器关键词"]
  }
}
```

---

## 规则语法

### 规则文件格式

规则文件是文本文件，每行一条规则，在系统中存储的格式如下：

```
expression\t#offset#description
```

**格式说明**：
- `expression`: 规则表达式（匹配条件）
- `#offset#`: 规则序号（从 0 开始，由系统自动生成）
- `description`: 规则描述（匹配时作为标签返回）

**导入时格式**：通过 `_import` API 导入时，只需提供 `expression` 和 `description`，系统会自动生成 offset：
```json
{
  "expression": "枪 or 手枪 or 步枪",
  "description": "涉枪武器关键词"
}
```

**系统存储示例**（导入后在 `.match_rules` 索引中的格式）：
```
枪 or 手枪 or 步枪	#0#涉枪武器关键词
AK47 or AK-47 or M16	#1#枪支型号
海洛因 or 冰毒 or 摇头丸	#2#毒品名称
```

**注意**：
- `#` 开头的行是注释
- 空行会被跳过
- 可使用 `\\` 作为续行符将长规则分成多行
- 表达式和描述之间使用 Tab 键分隔（`\t`）

### 基本语法

#### 1. 逻辑操作符

| 操作符 | 别名 | 优先级 | 说明 |
|--------|------|--------|------|
| `and` | `&` | 中 | 同时满足（逻辑与） |
| `or` | `\|` | 低 | 满足任一（逻辑或） |
| `not` | `&!`, `-` | 中（二元）/最高（一元） | 不包含（逻辑非） |
| `near` | 无 | 高 | 距离匹配 |

**示例**：
```
# AND: 同时包含两个词
中国 and 人民

# OR: 包含任一词
中国 or 人民

# NOT: 包含前者但不包含后者
中国 and not 日本
中国 not 日本
中国 - 日本
```

#### 2. 关键词匹配

```
枪 or 手枪 or 步枪
```

匹配包含任一关键词的文档。

**⚠️ 特殊字符警告：减号 `-` 的特殊含义**

减号 `-` 是 `not` 操作符的别名，会被解释为逻辑非。如果要匹配包含减号的字面文本，**必须用引号括起来**。

```
# ❌ 错误：会被解释为"包含中国 AND 不包含2025年GDP"
中国-2025年GDP

# ✅ 正确：匹配字面文本"中国-2025年GDP"
"中国-2025年GDP"

# ❌ 错误：会被解释为"包含高端 AND 不包含低端"
高端-低端产品

# ✅ 正确：匹配字面文本"高端-低端产品"
"高端-低端产品"
```

**其他需要加引号的场景**：
- 包含空格：`"紧急 通知"`
- 包含特殊符号：`"C++编程"`, `"@用户名"`
- 包含操作符关键字：`"and或or"`

#### 3. 字段限定

**语法**：
- `fieldname(expression)`: 子串匹配，在指定字段的任意位置匹配表达式
- `fieldname[expression]`: 全匹配，表达式必须匹配字段的全部内容

**示例**：
```
# 在 title 字段中查找"紧急"
title(紧急)

# 在 author 字段中查找"张三"
author(张三)

# 组合使用
title(紧急) and content(通知) and category(安全)

# 全匹配（用于精确匹配）
status[已审核]
```

**字段处理机制**：

规则引擎对字段的处理遵循以下规则：

1. **默认字段**：`title` 和 `content` 是系统内置字段，无需声明
   - 未指定字段的表达式默认匹配 `content` 字段

2. **未声明字段的处理**：文档中未在编译时声明的字段，会被当作 `content` 字段处理
   - 例如：`author(张三)` 中的 `author` 如果未声明，会按 `content` 字段匹配

3. **数值字段**：用于范围匹配的数值字段（如 `range(100, 500)`）**必须**在编译时通过 `fields` 参数声明
   - 未声明的数值字段会导致编译失败

**字段声明示例**：
```json
# 编译时声明数值字段
POST /_match_rules/repo/_compile
{
  "fields": ["range", "price", "ram_gb"]
}

# 规则中使用
{
  "expression": "title(手机) and price(3000, 5000)"
}
```

#### 4. 正则表达式

**简单正则**（用 `{{...}}` 括起来）：

```
# 匹配北京电话号码
{{(\(010\)|010)-?[0-9]{8}}}

# 匹配邮箱地址
{{\w+@\w+\.\w+}}
```

**复合正则**（用 `[...]` 括起来，支持逻辑运算）：

```
# 匹配除 .com 外的所有网站
[{{https?://([\w\d_-]+\.)+[\w\d_-]+}} and not {{.*\.com}}]

# 字段中使用正则
author({{张.*}})
```

**重要**：正则表达式必须用**双引号**包围。

❌ 错误：`author(/张.*/)`
✅ 正确：`author("/张.*/")`

### 高级功能

#### 距离匹配 (near)

`near` 用于匹配两个词之间的距离。

**语法**：
- `A near/num B`: A 和 B 之间距离不超过 num（不分前后）
- `A near/+num B`: A 在 B 之前，距离不超过 num
- `A near/-num B`: A 在 B 之后，距离不超过 num

**距离单位**：
- 中文：每个汉字为 1 个单位
- 英文/数字：每个单词为 1 个单位（如 `computer`、`10086`、`varname_123` 均为 1 个单位）

**示例**：

```
# "中国"和"人民"之间距离不超过 3
中国 near/3 人民

# "中国"必须在"人民"之前，距离不超过 5
中国 near/+5 人民

# 链式 near（等价于多个 near 的 and）
中国 near/3 人民 near/5 勤劳 near/7 勇敢
```

**复合 near**：
```
(中国 or 中华 or 华夏) near/3 (人民 or 百姓) near/5 (勤劳 or 勤奋)
```

#### 数值范围匹配

```
range(400, 600)
price(1000, 5000)
```

匹配数值在指定范围内的文档（闭区间 [min, max]）。

**要求**：
1. 字段类型必须是数值
2. 编译时必须声明字段名（使用 `fields` 参数）

**示例**：
```json
POST /_match_rules/opinion_check/_import
{
  "rules": [
    {
      "expression": "title(威马M5) and content(续航) and range(0, 500)",
      "description": "威马M5续航不足"
    }
  ]
}

POST /_match_rules/opinion_check/_compile
{
  "fields": ["range", "price"]
}
```

### 规则编写最佳实践

#### 1. 从简单到复杂

```
# 简单规则
枪 or 手枪

# 中等复杂度
title(枪支) and (买卖 or 交易 or 出售)

# 复杂规则
(枪 or 手枪 or 步枪) near/5 (买 or 卖 or 交易) and not (玩具 or 模型)
```

#### 2. 合理使用字段限定

```
# 标题匹配更精准
title(紧急通知)

# 组合多个字段
title(安全) and content(漏洞) and author(安全团队)
```

#### 3. 使用 near 提高精确度

```
# 简单 OR 可能误匹配
枪支 or 交易

# 使用 near 限制距离
枪支 near/10 交易
```

#### 4. 善用 NOT 排除干扰

```
# 排除玩具枪
(枪 or 手枪) and not (玩具 or 模型 or 仿真)
```

#### 5. 正则表达式用于模糊匹配

```
# 匹配所有以"张"开头的姓名
author({{张.*}})

# 匹配手机号
{{1[3-9][0-9]{9}}}
```

### 常见错误

#### ❌ 错误 1: 减号未加引号导致误解析

```
# 错误：会被解释为 NOT 操作
title(2024-2025年报)
# 实际匹配：title 包含"2024"但不包含"2025年报"

# 正确：匹配字面文本
title("2024-2025年报")
```

**常见场景**：
- 日期范围：`"2024-01-01"`, `"2024-2025"`
- 产品型号：`"iPhone-15"`, `"GTX-4090"`
- 复合词：`"高端-低端"`, `"线上-线下"`

#### ❌ 错误 2: 正则未加引号

```
# 错误
author(/张.*/)

# 正确
author("/张.*/")
```

#### ❌ 错误 3: 忘记声明自定义字段

```json
// 规则中使用了 range 字段
"expression": "range(100, 500)"

// 但编译时忘记声明
POST /_match_rules/repo/_compile
{
  // ❌ 缺少 fields 参数
}

// ✅ 正确做法
POST /_match_rules/repo/_compile
{
  "fields": ["range"]
}
```

#### ❌ 错误 4: 表达式和描述未用 Tab 分隔

```
# 错误（使用空格）
枪 or 手枪 涉枪内容

# 正确（使用 Tab）
枪 or 手枪	涉枪内容
```

### 特殊字符速查表

| 字符 | 在规则中的含义 | 如何匹配字面值 | 示例 |
|------|----------------|----------------|------|
| `-` | NOT 操作符 | 用引号括起来 | `"2024-2025"` |
| `&` | AND 操作符 | 用引号括起来 | `"Tom&Jerry"` |
| `\|` | OR 操作符 | 用引号括起来 | `"A\|B"` |
| `(` `)` | 分组、字段限定 | 用引号括起来 | `"(重要)"` |
| `[` `]` | 全匹配、复合正则 | 用引号括起来 | `"[TODO]"` |
| `{` `}` | 正则表达式 | 用引号括起来 | `"{变量}"` |
| 空格 | 词分隔符 | 用引号括起来 | `"hello world"` |

**建议**：当规则包含任何可能引起歧义的字符时，养成加引号的习惯。

---

## 高级应用示例

### 示例 1: 舆情监控

使用数值范围匹配来监控产品评价中的数值异常。

**导入规则**：
```json
POST /_match_rules/opinion_check/_import
{
  "name": "舆情监控规则",
  "rules": [
    {
      "expression": "title(威马M5) and content(续航) and range(0, 500)",
      "description": "威马M5续航不足"
    },
    {
      "expression": "title(威马M7) and content(续航) and range(2500, 9999)",
      "description": "威马M7续航异常高"
    }
  ]
}
```

**编译时声明字段**：
```json
POST /_match_rules/opinion_check/_compile
{
  "fields": ["range", "price", "energy_kwh"],
  "quiet": true
}
```

**说明**：编译时只需声明规则中**实际使用**的数值字段。上述规则中使用了 `range(0, 500)` 和 `price(2500, 9999)`，所以声明了这两个字段。

**Pipeline 配置**：
```json
PUT _ingest/pipeline/opinion_check
{
  "description": "使用规则引擎监测商家网络舆情",
  "processors": [
    {
      "check_match_rules": {
        "id": "opinion_check",
        "fields": [
          "range",
          "price",
          "energy_kwh",
          "fuel_lper100km",
          "ram_gb",
          "storage_gb",
          "volume_ml",
          "concentration"
        ]
      }
    }
  ]
}
```

**说明**：Pipeline 中的 `fields` 参数可以包含更多数值字段，以便支持文档中可能出现的其他数值字段匹配。这些字段不需要在当前规则库中使用，但如果文档包含这些字段，规则引擎会正确处理。

**测试匹配**：
```json
POST /_ingest/pipeline/opinion_check/_simulate
{
  "docs": [
    {
      "_index": "reviews",
      "_id": "doc_1",
      "_source": {
        "title": "威马M5续航测试",
        "content": "威马M5的续航表现一般，实测续航里程偏低",
        "range": 450
      }
    },
    {
      "_index": "reviews",
      "_id": "doc_2",
      "_source": {
        "title": "威马M7续航表现",
        "content": "威马M7的续航非常出色",
        "range": 2650
      }
    }
  ]
}
```

**预期结果**：
```json
{
  "docs": [
    {
      "doc": {
        "_source": {
          "title": "威马M5续航测试",
          "content": "威马M5的续航表现一般，实测续航里程偏低",
          "range": 450,
          "tags": ["#0#威马M5续航不足"]
        }
      }
    },
    {
      "doc": {
        "_source": {
          "title": "威马M7续航表现",
          "content": "威马M7的续航非常出色",
          "range": 2650,
          "tags": ["#1#威马M7续航异常高"]
        }
      }
    }
  ]
}
```

---

## 规则索引结构

### .match_rules 索引说明

`.match_rules` 是 Rules 插件使用的内部索引（名称以 `.` 开头），用于存储规则库的元数据和配置信息。每个规则库对应索引中的一条文档。

**索引特性**：
- **索引名称**：`.match_rules`（以 `.` 开头；当前初始化逻辑未显式设置 `index.hidden`）
- **分片配置**：1 个主分片，1 个副本
- **自动扩展**：副本数自动扩展到 0-1
- **文档 ID**：使用 `repo_id` 作为文档 ID

**核心字段**：

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `name` | text | ✅ | 规则库名称 |
| `description` | text | ❌ | 规则库描述 |
| `tags` | keyword[] | ❌ | 规则库标签（用于分类） |
| `rules` | text | ✅ | 规则文本内容（格式：`expression\t#offset#description`） |
| `version` | long | ✅ | 规则库版本（自动递增：1, 2, 3...） |
| `status` | keyword | ✅ | 编译状态：`pending`/`compiled`/`partial`/`failed` |
| `total_rules` | integer | ❌ | 编译后的规则总数（导入后可能为空） |
| `compiled_at` | date | ❌ | 编译完成时间 |
| `compiled_nodes` | keyword[] | ❌ | 已编译节点列表 |
| `fields` | keyword[] | ❌ | 编译时声明的字段列表 |
| `composite` | keyword[] | ❌ | 联合索引定义列表（2026-01-18 新增） |
| `error_message` | text | ❌ | 错误信息（编译失败时） |
| `created` | date | ✅ | 创建时间 |
| `updated` | date | ✅ | 更新时间 |

**规则文本格式**：

`rules` 字段存储完整的规则定义，每行一条规则，包含自动生成的 offset：

```
expression\t#offset#description
```

其中：
- `offset` 是系统自动生成的规则序号（从 0 开始）
- 通过 `_import` API 导入时**不需要**手动指定 offset

示例：
```
枪 or 手枪 or 步枪\t#0#涉枪武器关键词
AK47 or AK-47 or M16\t#1#枪支型号
海洛因 or 冰毒\t#2#毒品名称
```

**查询示例**：

```json
# 查询单个规则库（排除 rules 字段避免大量数据）
GET .match_rules/_doc/security_v1?_source_excludes=rules

# 搜索所有已编译的规则库
GET .match_rules/_search
{
  "query": {
    "term": {
      "status": "compiled"
    }
  },
  "_source": {
    "excludes": ["rules"]
  }
}

# 查询特定标签的规则库
GET .match_rules/_search
{
  "query": {
    "term": {
      "tags": "security"
    }
  }
}
```

---

## API 参考

### 导入规则 API

支持以下 4 条导入路由：

- `POST /_match_rules/{repo_id}/_import`：覆盖导入（upsert）
- `PUT /_match_rules/{repo_id}/_import`：追加导入（append）
- `POST /_match_rules/_import`：自动生成 `repo_id` 后覆盖导入
- `PUT /_match_rules/_import`：自动生成 `repo_id` 后追加导入

#### POST /_match_rules/{repo_id}/_import

覆盖导入（删除旧规则，创建新规则）。

**请求体**：
```json
{
  "name": "规则库名称",
  "description": "规则库描述",
  "tags": ["tag1", "tag2"],
  "rules": [
    {
      "expression": "规则表达式",
      "description": "规则描述"
    }
  ]
}
```

#### PUT /_match_rules/{repo_id}/_import

追加导入（保留旧规则，添加新规则）。

**示例**：
```json
PUT /_match_rules/security_v1/_import
{
  "rules": [
    {
      "expression": "买枪 or 卖枪 or 枪支交易",
      "description": "武器交易"
    }
  ]
}
```

**响应**：
```json
{
  "success": true,
  "message": "Successfully imported 4 rules to repo_id 'security_v1' (version: 2)",
  "repo_id": "security_v1",
  "version": "2",
  "rule_count": 4
}
```

#### POST /_match_rules/_import

自动生成 repo_id（格式：`repo_{timestamp}`）。

#### PUT /_match_rules/_import

自动生成 repo_id（格式：`repo_{timestamp}`）并使用追加语义导入。

**说明**：
- 由于每次请求都会生成新的 `repo_id`，该路由在多数场景下等价于创建新规则库文档
- 若需要对同一规则库执行真实追加，请使用 `PUT /_match_rules/{repo_id}/_import`

**导入后的状态变化（适用于 POST/PUT）**：
- 每次导入都会重置编译状态为 `pending`
- 导入后需要重新调用 `POST /_match_rules/{repo_id}/_compile` 才会让新规则生效

### 编译规则 API

#### POST /_match_rules/{repo_id}/_compile

编译规则库为二进制格式。

**请求体**：
```json
{
  "fields": ["author", "source", "range"],
  "quiet": true
}
```

**注意**：
- 请求体是必需的，至少传空对象 `{}`（不带 body 会返回解析错误）

**参数说明**：

| 参数 | 类型 | 必需 | 默认值 | 说明 |
|------|------|------|--------|------|
| `fields` | array | ❌ | [] | 编译时声明的字段列表（用于字段限定与范围匹配等） |
| `composite` | array | ❌ | [] | 联合索引定义（用于多字段组合匹配，提升性能） |
| `quiet` | boolean | ❌ | true | 是否静默模式（不输出详细日志） |

**联合索引**：

`composite` 参数用于定义多字段组合索引，提升多字段 AND 条件的匹配性能：

```json
{
  "fields": ["gender", "age", "income"],
  "composite": [
    "(gender,age,income)",           // 基本格式
    "s{:}(name,city,province)"       // 带分隔符格式
  ]
}
```

**格式说明**：
- `(field1,field2,field3)`: 基本格式，多个字段组合索引
- `s{sep}(field1,field2)`: 带分隔符格式，`sep` 为自定义分隔符（如 `:`）

**性能优势**：
- 单字段索引需要分别扫描 + 内存交集
- 联合索引一次扫描组合索引，性能提升 3-10 倍

**多节点集群**：编译会自动广播到所有 Ingest 节点；若个别节点未安装 rules 插件或编译失败，结果可能为部分成功（`partial`）。

**响应**：
```json
{
  "success": true,
  "message": "Broadcast compile completed for 3 nodes:\n  ✓ node-1: 15000 rules (2500ms)\n  ✓ node-2: 15000 rules (2480ms)\n  ✓ node-3: 15000 rules (2520ms)",
  "result": {
    "duration_ms": 2520,
    "total_rules": 15000,
    "indexed_rules": 0
  }
}
```

### 查询规则库 API

#### GET .match_rules/_doc/{repo_id}

查询规则库详情。

```json
GET .match_rules/_doc/security_v1
```

**响应**：
```json
{
  "_index": ".match_rules",
  "_id": "security_v1",
  "_version": 2,
  "found": true,
  "_source": {
    "name": "安全规则库",
    "description": "用于安全事件分类的规则库",
    "tags": ["security", "content-filter"],
    "rules": "枪 or 手枪 or 步枪 or 猎枪\t#0#涉枪武器关键词\nAK47 or AK-47 or M16 or M4A1\t#1#枪支型号\n...",
    "version": "2",
    "status": "compiled",
    "compiled_at": "2026-01-07T08:00:00Z",
    "total_rules": 4,
    "compiled_nodes": ["node-1", "node-2", "node-3"],
    "created": "2025-12-31T10:00:00Z",
    "updated": "2025-12-31T10:15:00Z"
  }
}
```

**字段说明**：

`.match_rules` 索引用于存储规则库的元数据和配置信息。每个规则库对应一条文档，文档 ID 即为 `repo_id`。

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | text | 规则库名称（用户自定义，用于标识规则库用途） |
| `description` | text | 规则库描述（详细说明规则库的作用和使用场景） |
| `tags` | keyword[] | 规则库标签（用于分类和检索，如 ["security", "content-filter"]） |
| `rules` | text | 规则文本内容（完整的规则定义，每行一条规则，格式：`expression\t#offset#description`） |
| `version` | long | 规则库版本（每次导入递增，格式：1, 2, 3...） |
| `status` | keyword | 编译状态：`pending`（待编译）、`compiled`（已编译）、`partial`（部分编译）、`failed`（失败） |
| `total_rules` | integer | 编译后的规则总数（未编译时可能为空） |
| `compiled_at` | date | 编译完成时间（最后一次成功编译的时间戳） |
| `compiled_nodes` | keyword[] | 已编译节点列表（已成功编译此规则库的节点名称） |
| `fields` | keyword[] | 编译时声明的字段列表（用于字段限定与范围匹配等） |
| `composite` | keyword[] | 联合索引定义列表（用于多字段组合匹配，如 ["(price,range)", "(title,content)"]） |
| `error_message` | text | 错误信息（编译失败时的详细错误描述） |
| `created` | date | 创建时间（规则库首次导入的时间） |
| `updated` | date | 更新时间（规则库最后一次修改的时间） |

**注意**：
- `rules` 字段可能包含大量文本（数万条规则），查询时建议使用 `_source_excludes=rules` 排除此字段
- `version` 字段在每次使用 `POST /_match_rules/{repo_id}/_import`（覆盖模式）或 `PUT /_match_rules/{repo_id}/_import`（追加模式）导入时自动递增
- `compiled_nodes` 字段在多节点集群中记录了所有成功编译的节点，用于判断集群同步状态

### 删除规则库 API

#### DELETE /_match_rules/{repo_id}

删除规则库（包括索引文档、物理文件和本地元数据）。

**HTTP 状态码**：
- `200`：删除成功（`success: true`）
- `500`：业务失败（如存在 Pipeline 依赖、索引或物理文件删除失败，`success: false`）

**功能特性**：
- ✅ Pipeline 依赖检查：防止删除正在使用的规则库
- ✅ 集群广播删除：多节点环境下同步删除物理文件
- ✅ 物理文件清理：递归删除规则库目录
- ✅ 元数据更新：自动更新本地元数据文件

**依赖检查**：

删除前会检查是否有 Pipeline 正在使用该规则库，如果有依赖则删除失败：

```json
DELETE /_match_rules/security_v1
```

**失败响应（有依赖）**：
```json
{
  "success": false,
  "message": "Cannot delete rule repository: used by pipelines: security-check, fraud-detection",
  "node_name": "node-1"
}
```

**成功响应**：
```json
{
  "success": true,
  "message": "Deleted successfully",
  "node_name": "node-1"
}
```

**删除流程**：
1. 检查 Pipeline 依赖（递归检查嵌套 processors）
2. 删除索引文档（从 `.match_rules` 索引）
3. 广播删除到所有 Ingest 节点
4. 每个节点删除物理目录 + 更新元数据

**安全机制**：
- 路径安全验证（防止 `../` 遍历攻击）
- 超时控制（广播操作超时 5 分钟）

---

## 节点启动与规则同步

### 自动同步机制

节点启动时，Rules 插件会自动检查并同步缺失的规则库：

1. **检查本地规则库**：对比 `.match_rules` 索引中的规则库
2. **自动编译缺失库**：对于本地不存在的规则库，自动执行编译
3. **更新节点状态**：同步完成后更新 `compiled_nodes` 列表

**预期行为**：
- 首次启动：自动编译所有规则库（通常 10-20 秒）
- 集群重启：验证本地规则库，如有缺失自动恢复
- 新节点加入：后台静默同步，不影响集群服务

### 同步期间的写入行为

在规则库同步期间，写入行为取决于集群拓扑与节点状态：部分场景会加同步 block 并出现重试等待，其他场景可能不加 block。

**用户体验**：
- 可能出现写入等待/重试（由重试策略决定），也可能直接由其他可用 Ingest 节点处理
- 等待时间取决于规则库大小与节点数量（通常几秒到几分钟）
- 若超时或异常，写入仍可能失败，需按错误信息重试

**不受影响的操作**：
- ✅ 所有读操作（查询、获取文档等）
- ✅ 系统索引的写入（`.match_rules` 等）

**预计等待时间**：
- 通常 10-20 秒
- 大型规则库（数万条）可能需要更长时间

---

## 批量导入工具

对于大规则文件，可使用 Python 脚本批量导入：

```bash
# 基本用法（追加模式）
python import_rules.py -f rules.txt -r security_v1 -n "安全规则库"

# 覆盖模式（删除旧规则）
python import_rules.py -f rules.txt -r opinion_v9 -n "舆情规则库" --overwrite

# 指定名称、描述和标签
python import_rules.py -f rules.txt -r opinion_v9 \
  -n "舆情监控规则" \
  -d "用于舆情监控的规则库" \
  -t opinion -t monitoring

# 使用 HTTPS 和自定义认证
python import_rules.py -f rules.txt -r security_v1 -n "安全规则" \
  --url https://localhost:9200 \
  --username admin \
  --password admin123
```

**规则文件格式** (`rules.txt`)：

规则文件每行一条规则，使用 Tab 键分隔表达式和描述（**不需要手动指定 offset**）：

```
枪 or 手枪 or 步枪	涉枪武器关键词
AK47 or AK-47 or M16	枪支型号
海洛因 or 冰毒	毒品名称
```

**说明**：导入时系统会自动为每条规则生成 offset（从 0 开始），无需手动添加 `#offset#` 部分。

**工具位置**：`plugins/rules/src/test/resources/import_rules.py`

---

## 使用场景

### 1. 内容安全审核

**规则示例**：
```json
{
  "rules": [
    {
      "expression": "枪 or 手枪 or 步枪 or 猎枪",
      "description": "涉枪内容"
    },
    {
      "expression": "海洛因 or 冰毒 or 摇头丸",
      "description": "涉毒内容"
    },
    {
      "expression": "isis or 伊斯兰国 or IS组织",
      "description": "恐怖组织"
    }
  ]
}
```

**查询示例**：
```json
GET security-events/_search
{
  "query": {
    "terms": {
      "tags": ["#0#涉枪内容", "#1#涉毒内容"]
    }
  }
}
```

### 2. 新闻分类

**规则示例**：
```json
{
  "rules": [
    {
      "expression": "author(张三) and source(新华网)",
      "description": "张三_新华网"
    },
    {
      "expression": "title(紧急) and content(通知) and category(安全)",
      "description": "紧急安全通知"
    }
  ]
}
```

### 3. 商品推荐

**规则示例**：
```json
{
  "rules": [
    {
      "expression": "category(手机) and price(3000, 5000)",
      "description": "中端手机"
    },
    {
      "expression": "category(笔记本) and ram_gb(16, 32)",
      "description": "高性能笔记本"
    }
  ]
}
```

### 4. 舆情监控

**规则示例**（见上文"数值范围匹配"章节）

---

## 性能优化

### 规则库大小建议

- 单个规则库不超过 50,000 条规则
- 大规则库考虑拆分为多个 repo_id

### 编译性能参考

| 规则数 | 编译耗时 |
|--------|----------|
| 1,000 | ~100-200ms |
| 10,000 | ~500-1000ms |
| 100,000 | ~5-10s |

### 字段优化

- 只声明规则中实际使用的字段
- 数值字段（如 `range`、`price`）必须声明，否则编译失败

---

## 限制和注意事项

1. **License 要求**：需要启用 `rule-engine` 特性的有效 License
2. **系统配置要求**：必须设置 `bootstrap.system_call_filter: false`（见"环境要求"章节）
3. **目标字段可配置**：默认写入 `tags` 字段，可通过 `target_field` 参数自定义
4. **数值字段类型**：数值范围匹配要求字段值为数值类型
5. **正则语法**：必须使用双引号包围正则表达式
6. **编译依赖索引**：编译前必须先导入规则到 `.match_rules` 索引
7. **节点启动同步**：节点启动时会自动同步规则库，写入操作会等待同步完成（通常 10-20 秒）
8. **元数据文件**：`.compiled_repos.json` 用于检测文件丢失，请勿手动删除

---


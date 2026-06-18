---
title: "索引模板"
date: 0001-01-01
description: "索引模板的创建、管理和优化：自动预配 Settings/Mappings/Aliases，模板叠加与常见坑。"
summary: "索引模板 #  日志、指标、审计事件这类数据，常见模式是&quot;按天/按月一个新索引&quot;。如果每次新索引都要手动 PUT /index 配 settings + mappings，很快就会变成运维噩梦。
索引模板（Index Template） 就是为了解决这个问题：当新索引（手动或自动）被创建时，只要名字命中规则，就自动套用一组预配置——索引设置、映射和别名。
 先了解： 索引管理：创建、删除与重建索引 | 时间序列建模
 Easysearch 支持三类模板：
   类型 API 路径 说明     可组合索引模板 _index_template 推荐使用，支持组件模板组合   组件模板 _component_template 可复用的模板构建块，被可组合模板引用   遗留索引模板 _template 旧版 API，建议迁移到可组合模板     注意：可组合索引模板（_index_template）优先级高于遗留模板（_template）。如果同时存在匹配的可组合模板和遗留模板，将使用可组合模板。
  可组合索引模板 #  可组合索引模板是推荐的模板方式，支持通过 composed_of 引用组件模板，实现模板的模块化组合。
创建模板 #  PUT _index_template/daily_logs { &#34;index_patterns&#34;: [&#34;logs-2024-01-*&#34;], &#34;priority&#34;: 100, &#34;template&#34;: { &#34;aliases&#34;: { &#34;my_logs&#34;: {} }, &#34;settings&#34;: { &#34;number_of_shards&#34;: 2, &#34;number_of_replicas&#34;: 1 }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;timestamp&#34;: { &#34;type&#34;: &#34;date&#34;, &#34;format&#34;: &#34;yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis&#34; }, &#34;value&#34;: { &#34;type&#34;: &#34;double&#34; } } } } } 请求参数 #     参数 类型 描述 默认值     create boolean 为 true 时，仅在模板不存在时创建，不会覆盖已有模板 false   cause string 创建/更新模板的原因（记录到日志） —   master_timeout time 连接主节点的超时时间 30s    请求体参数 #     参数 类型 描述 必填     index_patterns array 索引名称匹配模式列表 是   template object 包含 aliases、settings、mappings 的模板定义 否   composed_of array 引用的组件模板名称列表，按顺序合并 否   priority number 模板优先级，值越大优先级越高 0   version number 模板版本号（用户自定义，仅供管理使用） —   _meta object 模板的元数据信息 —   data_stream object 设置后匹配的索引将作为数据流创建 —    查看模板 #  # 获取指定模板 GET _index_template/daily_logs # 获取所有可组合模板 GET _index_template # 通配符匹配 GET _index_template/daily* 查询参数 #     参数 类型 描述 默认值     flat_settings boolean 以扁平格式返回 Settings false   local boolean 从本地节点返回信息，不查询主节点 false   master_timeout time 连接主节点的超时时间 30s    检查模板是否存在 #  HEAD _index_template/daily_logs 返回 200 表示存在，404 表示不存在。"
---


# 索引模板

日志、指标、审计事件这类数据，常见模式是"按天/按月一个新索引"。如果每次新索引都要手动 `PUT /index` 配 settings + mappings，很快就会变成运维噩梦。

**索引模板（Index Template）** 就是为了解决这个问题：当新索引（手动或自动）被创建时，只要名字命中规则，就自动套用一组预配置——索引设置、映射和别名。

> 先了解：[索引管理：创建、删除与重建索引](./index-management.md) | [时间序列建模]({{< relref "/docs/best-practices/data-modeling/time-series" >}})

Easysearch 支持三类模板：

| 类型 | API 路径 | 说明 |
|------|----------|------|
| **可组合索引模板** | `_index_template` | 推荐使用，支持组件模板组合 |
| **组件模板** | `_component_template` | 可复用的模板构建块，被可组合模板引用 |
| **遗留索引模板** | `_template` | 旧版 API，建议迁移到可组合模板 |

> **注意**：可组合索引模板（`_index_template`）优先级高于遗留模板（`_template`）。如果同时存在匹配的可组合模板和遗留模板，将使用可组合模板。

---

## 可组合索引模板

可组合索引模板是推荐的模板方式，支持通过 `composed_of` 引用组件模板，实现模板的模块化组合。

### 创建模板

```json
PUT _index_template/daily_logs
{
  "index_patterns": ["logs-2024-01-*"],
  "priority": 100,
  "template": {
    "aliases": {
      "my_logs": {}
    },
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        },
        "value": {
          "type": "double"
        }
      }
    }
  }
}
```

#### 请求参数

| 参数 | 类型 | 描述 | 默认值 |
|------|------|------|--------|
| `create` | `boolean` | 为 `true` 时，仅在模板不存在时创建，不会覆盖已有模板 | `false` |
| `cause` | `string` | 创建/更新模板的原因（记录到日志） | — |
| `master_timeout` | `time` | 连接主节点的超时时间 | `30s` |

#### 请求体参数

| 参数 | 类型 | 描述 | 必填 |
|------|------|------|------|
| `index_patterns` | `array` | 索引名称匹配模式列表 | 是 |
| `template` | `object` | 包含 `aliases`、`settings`、`mappings` 的模板定义 | 否 |
| `composed_of` | `array` | 引用的组件模板名称列表，按顺序合并 | 否 |
| `priority` | `number` | 模板优先级，值越大优先级越高 | `0` |
| `version` | `number` | 模板版本号（用户自定义，仅供管理使用） | — |
| `_meta` | `object` | 模板的元数据信息 | — |
| `data_stream` | `object` | 设置后匹配的索引将作为数据流创建 | — |

### 查看模板

```json
# 获取指定模板
GET _index_template/daily_logs

# 获取所有可组合模板
GET _index_template

# 通配符匹配
GET _index_template/daily*
```

#### 查询参数

| 参数 | 类型 | 描述 | 默认值 |
|------|------|------|--------|
| `flat_settings` | `boolean` | 以扁平格式返回 Settings | `false` |
| `local` | `boolean` | 从本地节点返回信息，不查询主节点 | `false` |
| `master_timeout` | `time` | 连接主节点的超时时间 | `30s` |

### 检查模板是否存在

```json
HEAD _index_template/daily_logs
```

返回 `200` 表示存在，`404` 表示不存在。

### 删除模板

```json
DELETE _index_template/daily_logs
```

#### 查询参数

| 参数 | 类型 | 描述 |
|------|------|------|
| `timeout` | `time` | 操作超时时间 |
| `master_timeout` | `time` | 连接主节点的超时时间 |

### 模拟模板

在正式创建模板前，可以使用模拟 API 预览模板的合并效果。

#### 模拟已有模板

查看一个已有模板（或请求体中提供的模板定义）的最终合并结果：

```json
POST _index_template/_simulate/daily_logs
```

#### 模拟索引匹配

查看对于一个具体的索引名称，最终会应用哪些模板设置：

```json
POST _index_template/_simulate_index/logs-2024-01-15
```

也可以在请求体中提供一个新的模板定义，模拟它加入系统后的效果：

```json
POST _index_template/_simulate_index/logs-2024-01-15
{
  "index_patterns": ["logs-*"],
  "priority": 200,
  "template": {
    "settings": {
      "number_of_shards": 5
    }
  }
}
```

---

## 组件模板

组件模板是可复用的模板构建块，不能直接应用于索引，只能被可组合索引模板通过 `composed_of` 引用。

适用场景：多个索引模板需要共享相同的 Settings 或 Mappings 片段。

### 创建组件模板

```json
PUT _component_template/base_settings
{
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "index.refresh_interval": "5s"
    }
  }
}
```

```json
PUT _component_template/log_mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "message": { "type": "text" },
        "level": { "type": "keyword" },
        "host": { "type": "keyword" }
      }
    }
  }
}
```

#### 查询参数

| 参数 | 类型 | 描述 |
|------|------|------|
| `create` | `boolean` | 仅在不存在时创建 |
| `timeout` | `time` | 操作超时时间 |
| `master_timeout` | `time` | 连接主节点的超时时间 |

### 在索引模板中引用组件模板

```json
PUT _index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "priority": 100,
  "composed_of": ["base_settings", "log_mappings"],
  "template": {
    "aliases": {
      "logs": {}
    }
  }
}
```

`composed_of` 中的组件模板按顺序合并，后面的覆盖前面的同名设置。索引模板自身的 `template` 设置优先级最高，会覆盖所有组件模板的设置。

### 查看组件模板

```json
# 获取指定组件模板
GET _component_template/base_settings

# 获取所有组件模板
GET _component_template
```

### 检查组件模板是否存在

```json
HEAD _component_template/base_settings
```

### 删除组件模板

```json
DELETE _component_template/base_settings
```

> **注意**：删除正在被索引模板引用的组件模板会导致错误。请先解除引用关系。

---

## 模板优先级与合并规则

当一个索引名称匹配多个模板时，按以下规则处理：

1. **可组合模板优先**：如果同时匹配可组合模板和遗留模板，使用可组合模板
2. **priority 决定优先级**：多个可组合模板匹配时，选择 `priority` 最高的
3. **组件模板按顺序合并**：`composed_of` 列表中的组件模板从左到右合并，后面覆盖前面
4. **索引模板自身设置最高优先**：索引模板的 `template` 字段会覆盖组件模板的同名设置

**工程建议**：
- 让全局模板只做"底座"（通用的 refresh 策略、通用动态模板约束）
- 让业务模板决定分片、副本、映射与别名
- 任何"例外"用更窄的 `index_patterns` 覆盖，而不是在通用模板里堆 if/else

示例：

```json
# 组件模板：基础设置
PUT _component_template/base
{
  "template": {
    "settings": { "number_of_shards": 1 }
  }
}

# 组件模板：高性能设置
PUT _component_template/performance
{
  "template": {
    "settings": { "number_of_shards": 3, "refresh_interval": "10s" }
  }
}

# 索引模板：组合以上两个组件
PUT _index_template/my_template
{
  "index_patterns": ["myindex-*"],
  "composed_of": ["base", "performance"],
  "priority": 100,
  "template": {
    "settings": { "number_of_replicas": 2 }
  }
}
```

最终效果：`number_of_shards` = 3（performance 覆盖 base），`refresh_interval` = 10s，`number_of_replicas` = 2（索引模板自身设置）。

---

## 把别名写进模板

模板里配置别名非常实用，常见用法：

- `logs_write`：写入别名（通常只指向当前写索引）
- `logs_read`：读取别名（指向多个历史索引）

但注意：**模板只能"给新索引加别名"，不会自动把老索引从别名里移走**。

如果你想维持"只查最近 90 天"的 `logs_recent`：
- 新索引会自动加入 `logs_recent`
- 但 90 天前的老索引仍需要你定期从别名里移除，或直接删除索引

这属于"数据退役/保留策略"的范筹，见：[数据生命周期管理]({{< relref "/docs/features/data-retention/lifecycle" >}})。

---

## 遗留索引模板

遗留模板使用 `_template` API，功能较为有限，不支持组件模板组合。建议新项目使用可组合索引模板。

### 创建遗留模板

```json
PUT _template/legacy_logs
{
  "index_patterns": ["legacy-logs-*"],
  "order": 0,
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "@timestamp": { "type": "date" }
    }
  }
}
```

> **注意**：遗留模板使用 `order` 控制优先级（数值越大优先级越高），而可组合模板使用 `priority`。

### 查看遗留模板

```json
# 获取指定模板
GET _template/legacy_logs

# 获取所有遗留模板
GET _template

# 查看所有模板列表
GET _cat/templates
```

### 删除遗留模板

```json
DELETE _template/legacy_logs
```

---

## 常见坑

- **自动创建索引的风险**：拼错索引名也会自动创建新索引，导致数据散落。生产里建议至少做命名白名单或关闭自动创建。
- **动态映射的意外字段**：日志字段飘忽不定时，动态映射可能推断出错误类型（比如把数字当字符串），建议用 `dynamic_templates` 做兜底约束。
- **分片数拍脑袋**：模板里把 `number_of_shards` 设很大，后果通常是集群碎成渣。分片策略见：[扩缩容与分片]({{< relref "/docs/operations/cluster-admin/cluster" >}})。

---

## API 参考汇总

### 可组合索引模板

| 操作 | 方法 | 端点 |
|------|------|------|
| 创建/更新 | `PUT` / `POST` | `/_index_template/{name}` |
| 查看 | `GET` | `/_index_template` 或 `/_index_template/{name}` |
| 检查存在 | `HEAD` | `/_index_template/{name}` |
| 删除 | `DELETE` | `/_index_template/{name}` |
| 模拟模板 | `POST` | `/_index_template/_simulate/{name}` |
| 模拟索引 | `POST` | `/_index_template/_simulate_index/{name}` |

### 组件模板

| 操作 | 方法 | 端点 |
|------|------|------|
| 创建/更新 | `PUT` / `POST` | `/_component_template/{name}` |
| 查看 | `GET` | `/_component_template` 或 `/_component_template/{name}` |
| 检查存在 | `HEAD` | `/_component_template/{name}` |
| 删除 | `DELETE` | `/_component_template/{name}` |

### 遗留索引模板

| 操作 | 方法 | 端点 |
|------|------|------|
| 创建/更新 | `PUT` / `POST` | `/_template/{name}` |
| 查看 | `GET` | `/_template` 或 `/_template/{name}` |
| 检查存在 | `HEAD` | `/_template/{name}` |
| 删除 | `DELETE` | `/_template/{name}` |

---

## 相关文档

- [数据流]({{< relref "./data-streams" >}})：使用索引模板创建数据流
- [索引生命周期管理]({{< relref "/docs/features/data-retention/ilm" >}})：在模板中关联 ILM 策略
- [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}})：完整的数据保留策略体系



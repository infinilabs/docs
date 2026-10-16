---
title: "快照生命周期"
date: 0001-01-01
summary: "快照生命周期管理 #  快照生命周期管理（Snapshot Lifecycle Management, SLM）提供自动化的快照创建与清理能力。 通过配置 SLM 策略，您可以按照预定计划自动创建快照，并根据保留条件自动删除过期快照。
 策略配置参考 #  策略结构 #  { &#34;description&#34;: &#34;策略描述&#34;, &#34;creation&#34;: { ... }, &#34;deletion&#34;: { ... }, &#34;snapshot_config&#34;: { ... }, &#34;notification&#34;: { ... } } creation — 快照创建配置 #     参数 描述 类型 是否必需     schedule 创建快照的时间计划。 object 是   time_limit 创建快照的最大等待时间。 string 否    schedule — 时间计划 #  支持 cron 表达式或固定间隔两种格式："
---


# 快照生命周期管理

快照生命周期管理（Snapshot Lifecycle Management, SLM）提供自动化的快照创建与清理能力。
通过配置 SLM 策略，您可以按照预定计划自动创建快照，并根据保留条件自动删除过期快照。

---

## 策略配置参考

### 策略结构

```json
{
  "description": "策略描述",
  "creation": { ... },
  "deletion": { ... },
  "snapshot_config": { ... },
  "notification": { ... }
}
```

### creation — 快照创建配置

| 参数         | 描述                     | 类型     | 是否必需 |
| ------------ | ------------------------ | -------- | -------- |
| `schedule`   | 创建快照的时间计划。       | `object` | 是       |
| `time_limit` | 创建快照的最大等待时间。   | `string` | 否       |

#### schedule — 时间计划

支持 cron 表达式或固定间隔两种格式：

**Cron 格式**：

```json
{
  "cron": {
    "expression": "0 8 * * *",
    "timezone": "Asia/Shanghai"
  }
}
```

**固定间隔格式**：

```json
{
  "interval": {
    "start_time": 1685348095913,
    "period": 1,
    "unit": "Hours"
  }
}
```

### deletion — 快照删除配置

| 参数         | 描述                                              | 类型     | 是否必需 |
| ------------ | ------------------------------------------------- | -------- | -------- |
| `schedule`   | 执行删除检查的时间计划。省略时使用创建计划的时间。    | `object` | 否       |
| `condition`  | 触发删除的条件。                                    | `object` | 是       |
| `time_limit` | 删除快照的最大等待时间。                             | `string` | 否       |

#### condition — 删除条件

| 参数        | 描述                               | 类型   | 是否必需   | 验证                        |
| ----------- | ---------------------------------- | ------ | ---------- | --------------------------- |
| `max_age`   | 删除超过此年龄的快照。               | `string` | 条件必需 ¹ | —                           |
| `max_count` | 当快照总数超过此值时删除最旧的快照。  | `int`    | 条件必需 ¹ | 必须 > `min_count`           |
| `min_count` | 始终保留的最小快照数量。             | `int`    | 否         | 必须 > 0，默认 `1`           |

> ¹ `max_age` 和 `max_count` 中至少需要指定一个。

### snapshot_config — 快照配置

| 参数                     | 描述                                    | 类型     | 是否必需 |
| ------------------------ | --------------------------------------- | -------- | -------- |
| `repository`             | 快照仓库名称。                           | `string` | 是       |
| `date_format`            | 快照名称中的日期格式。                    | `string` | 否       |
| `date_format_timezone`   | 日期格式使用的时区。                      | `string` | 否       |
| `indices`                | 要包含在快照中的索引（支持通配符）。       | `string` | 否       |
| `ignore_unavailable`     | 是否忽略不可用的索引。                    | `string` | 否       |
| `include_global_state`   | 是否包含集群全局状态。                    | `string` | 否       |
| `partial`                | 是否允许部分快照。                        | `string` | 否       |
| `metadata`               | 快照元数据（任意键值对）。                 | `object` | 否       |

### notification — 通知配置（可选）

| 参数         | 描述             | 类型     | 是否必需 |
| ------------ | ---------------- | -------- | -------- |
| `channel`    | 通知渠道配置。    | `object` | 是       |
| `conditions` | 触发通知的事件。  | `object` | 否       |

#### 通知触发条件

| 参数                  | 描述                   | 类型      | 默认值  |
| --------------------- | ---------------------- | --------- | ------- |
| `creation`            | 创建快照时通知。        | `boolean` | `true`  |
| `deletion`            | 删除快照时通知。        | `boolean` | `false` |
| `failure`             | 操作失败时通知。        | `boolean` | `false` |
| `time_limit_exceeded` | 超过时间限制时通知。     | `boolean` | `false` |

---

## 创建策略 {#create-policy}

创建一个新的快照管理策略。

#### 请求

```
POST /_slm/policies/{policyName}
```

#### 请求示例

以下策略配置：
- 每天上午 8 点自动创建快照，名称格式为 `yyyy-MM-dd-HH:mm`，存储在 `my_backup` 仓库
- 每天凌晨 1 点检查并删除超过 7 天的快照，保留至少 7 个，最多 21 个
- 创建和删除的超时限制均为 1 小时

```json
POST /_slm/policies/daily-policy
{
  "description": "每日快照策略",
  "creation": {
    "schedule": {
      "cron": {
        "expression": "0 8 * * *",
        "timezone": "Asia/Shanghai"
      }
    },
    "time_limit": "1h"
  },
  "deletion": {
    "schedule": {
      "cron": {
        "expression": "0 1 * * *",
        "timezone": "Asia/Shanghai"
      }
    },
    "condition": {
      "max_age": "7d",
      "max_count": 21,
      "min_count": 7
    },
    "time_limit": "1h"
  },
  "snapshot_config": {
    "date_format": "yyyy-MM-dd-HH:mm",
    "date_format_timezone": "Asia/Shanghai",
    "indices": "*",
    "repository": "my_backup",
    "ignore_unavailable": "true",
    "include_global_state": "false",
    "partial": "true",
    "metadata": {
      "any_key": "any_value"
    }
  }
}
```

#### 响应示例

```json
{
  "_id": "daily-policy-sm-policy",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "sm_policy": {
    "name": "daily-policy",
    "description": "每日快照策略",
    "schema_version": 17,
    "creation": {
      "schedule": {
        "cron": {
          "expression": "0 8 * * *",
          "timezone": "Asia/Shanghai"
        }
      },
      "time_limit": "1h"
    },
    "deletion": {
      "schedule": {
        "cron": {
          "expression": "0 1 * * *",
          "timezone": "Asia/Shanghai"
        }
      },
      "condition": {
        "max_age": "7d",
        "min_count": 7,
        "max_count": 21
      },
      "time_limit": "1h"
    },
    "snapshot_config": {
      "indices": "*",
      "metadata": {
        "any_key": "any_value"
      },
      "ignore_unavailable": "true",
      "date_format_timezone": "Asia/Shanghai",
      "include_global_state": "false",
      "date_format": "yyyy-MM-dd-HH:mm",
      "repository": "my_backup",
      "partial": "true"
    },
    "schedule": {
      "interval": {
        "start_time": 1685348095913,
        "period": 1,
        "unit": "Minutes"
      }
    },
    "enabled": true,
    "last_updated_time": 1685348095938,
    "enabled_time": 1685348095909
  }
}
```

---

## 获取策略 {#get-policy}

#### 获取单个策略

```
GET /_slm/policies/{policyName}
```

#### 响应示例

```json
{
  "_id": "daily-policy-sm-policy",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "sm_policy": {
    "name": "daily-policy",
    "description": "每日快照策略",
    "schema_version": 17,
    "creation": { "..." : "..." },
    "deletion": { "..." : "..." },
    "snapshot_config": { "..." : "..." },
    "schedule": { "..." : "..." },
    "enabled": true,
    "last_updated_time": 1685348095938,
    "enabled_time": 1685348095909
  }
}
```

#### 获取所有策略 {#list-policies}

```
GET /_slm/policies
```

| 查询参数      | 描述           | 类型     | 默认值 |
| ------------- | -------------- | -------- | ------ |
| `size`        | 每页返回数量。   | `int`    | `20`   |
| `from`        | 分页起始偏移量。 | `int`    | `0`    |
| `sortField`   | 排序字段。      | `string` | —      |
| `sortOrder`   | 排序方向。      | `string` | `asc`  |
| `queryString` | 搜索过滤条件。  | `string` | `*`    |

#### 响应示例

```json
{
  "policies": [
    {
      "_id": "daily-policy-sm-policy",
      "_seq_no": 0,
      "_primary_term": 1,
      "sm_policy": { "..." : "..." }
    }
  ],
  "total_policies": 1
}
```

---

## 更新策略 {#update-policy}

更新已有策略。必须指定 `if_seq_no` 和 `if_primary_term` 进行乐观并发控制。

#### 请求

```
PUT /_slm/policies/{policyName}?if_seq_no=0&if_primary_term=1
```

请求体格式与创建策略相同。

#### 请求示例

```json
PUT /_slm/policies/daily-policy?if_seq_no=0&if_primary_term=1
{
  "description": "每日快照策略（更新）",
  "creation": {
    "schedule": {
      "cron": {
        "expression": "0 9 * * *",
        "timezone": "Asia/Shanghai"
      }
    },
    "time_limit": "1h"
  },
  "deletion": {
    "schedule": {
      "cron": {
        "expression": "0 1 * * *",
        "timezone": "Asia/Shanghai"
      }
    },
    "condition": {
      "max_age": "7d",
      "max_count": 21,
      "min_count": 7
    },
    "time_limit": "1h"
  },
  "snapshot_config": {
    "date_format": "yyyy-MM-dd-HH:mm",
    "date_format_timezone": "Asia/Shanghai",
    "indices": "*",
    "repository": "my_backup",
    "ignore_unavailable": "true",
    "include_global_state": "false",
    "partial": "true"
  }
}
```

---

## 查看策略状态 {#explain}

使用 `_explain` API 查看策略的当前执行状态，包括下次创建/删除快照的时间，以及最近一次执行的结果。

支持通配符匹配多个策略。

#### 请求

```
GET /_slm/policies/{policyName}/_explain
```

#### 请求示例

```
GET /_slm/policies/daily*/_explain
```

#### 响应示例

```json
{
  "policies": [
    {
      "name": "daily-policy",
      "creation": {
        "current_state": "CREATION_START",
        "trigger": {
          "time": 1685404800000
        }
      },
      "deletion": {
        "current_state": "DELETION_START",
        "trigger": {
          "time": 1685379600000
        }
      },
      "policy_seq_no": 0,
      "policy_primary_term": 1,
      "enabled": true
    }
  ]
}
```

#### 状态说明

**创建工作流状态**：

| 状态                   | 描述                    |
| ---------------------- | ----------------------- |
| `CREATION_START`       | 等待触发条件。           |
| `CREATION_CONDITION_MET` | 条件已满足，准备创建。  |
| `CREATING`             | 正在创建快照。           |
| `CREATION_FINISHED`    | 创建完成。               |

**删除工作流状态**：

| 状态                   | 描述                    |
| ---------------------- | ----------------------- |
| `DELETION_START`       | 等待触发条件。           |
| `DELETION_CONDITION_MET` | 条件已满足，准备删除。  |
| `DELETING`             | 正在删除快照。           |
| `DELETION_FINISHED`    | 删除完成。               |

**执行状态**：

| 状态                  | 描述               |
| --------------------- | ------------------ |
| `IN_PROGRESS`         | 执行中。            |
| `RETRYING`            | 正在重试。          |
| `SUCCESS`             | 执行成功。          |
| `FAILED`              | 执行失败。          |
| `TIME_LIMIT_EXCEEDED` | 超过时间限制。       |

---

## 删除策略 {#delete-policy}

删除指定的 SLM 策略。

#### 请求

```
DELETE /_slm/policies/{policyName}
```

#### 请求示例

```
DELETE /_slm/policies/daily-policy
```

#### 响应示例

```json
{
  "_index": ".easysearch-ilm-config",
  "_type": "_doc",
  "_id": "daily-policy-sm-policy",
  "_version": 2,
  "result": "deleted",
  "forced_refresh": true,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 8,
  "_primary_term": 1
}
```

---

## 启动策略 {#start-policy}

启动（启用）一个已停止的策略。

#### 请求

```
POST /_slm/policies/{policyName}/_start
```

#### 响应示例

```json
{
  "acknowledged": true
}
```

---

## 停止策略 {#stop-policy}

停止（禁用）一个正在运行的策略。策略停止后不会自动创建或删除快照。

#### 请求

```
POST /_slm/policies/{policyName}/_stop
```

#### 响应示例

```json
{
  "acknowledged": true
}
```

---

## 完整示例

### 场景：每小时快照 + 7 天保留

```json
POST /_slm/policies/hourly-backup
{
  "description": "每小时快照，保留 7 天",
  "creation": {
    "schedule": {
      "cron": {
        "expression": "0 * * * *",
        "timezone": "Asia/Shanghai"
      }
    },
    "time_limit": "1h"
  },
  "deletion": {
    "schedule": {
      "cron": {
        "expression": "0 2 * * *",
        "timezone": "Asia/Shanghai"
      }
    },
    "condition": {
      "max_age": "7d",
      "max_count": 200,
      "min_count": 24
    },
    "time_limit": "1h"
  },
  "snapshot_config": {
    "date_format": "yyyy-MM-dd-HH:mm",
    "date_format_timezone": "Asia/Shanghai",
    "indices": "logs-*,metrics-*",
    "repository": "my_backup",
    "ignore_unavailable": "true",
    "include_global_state": "false",
    "partial": "true"
  }
}
```

### 检查策略状态

```
GET /_slm/policies/hourly-backup/_explain
```

### 停止策略（维护期间）

```
POST /_slm/policies/hourly-backup/_stop
```

### 恢复策略

```
POST /_slm/policies/hourly-backup/_start
```


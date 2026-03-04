---
title: "索引生命周期"
date: 0001-01-01
summary: "索引生命周期管理 #  索引生命周期管理（Index Lifecycle Management, ILM）为您提供了一种集成化、自动化的方式来高效管理时序数据。 通过配置 ILM 策略，您可以根据性能、可用性与数据保留需求，自动执行索引的滚动、归档和清理等操作。
 从 1.15.2 版本开始，index-management 已经成为 modules 的一部分，不需要单独安装插件。
 典型应用场景 #   自动滚动生成新索引：当现有索引达到指定大小或文档数量时，自动创建新索引。 周期性轮换索引：按天、周或月创建新索引，并将历史索引归档。 强制数据保留策略：自动删除过期索引，确保合规与存储成本可控。 分层存储：将热数据分配到高性能节点，冷数据迁移到廉价存储节点。   策略管理 API #  创建策略 #  创建一个新的生命周期策略。
请求 #  PUT _ilm/policy/{policyID}    参数 描述 类型 是否必需     policyID 策略 ID。 string 是    查询参数 #     参数 描述 类型 默认值     if_seq_no 仅当匹配此序列号时更新。 long —   if_primary_term 仅当匹配此主分片任期时更新。 long —    请求体 #  策略支持 ES 兼容的 phases 格式。系统内部会自动转换为 states 格式。"
---


# 索引生命周期管理

索引生命周期管理（Index Lifecycle Management, ILM）为您提供了一种集成化、自动化的方式来高效管理时序数据。
通过配置 ILM 策略，您可以根据性能、可用性与数据保留需求，自动执行索引的滚动、归档和清理等操作。

> 从 1.15.2 版本开始，index-management 已经成为 modules 的一部分，不需要单独安装插件。

## 典型应用场景

- **自动滚动生成新索引**：当现有索引达到指定大小或文档数量时，自动创建新索引。
- **周期性轮换索引**：按天、周或月创建新索引，并将历史索引归档。
- **强制数据保留策略**：自动删除过期索引，确保合规与存储成本可控。
- **分层存储**：将热数据分配到高性能节点，冷数据迁移到廉价存储节点。

---

## 策略管理 API

### 创建策略 {#create-policy}

创建一个新的生命周期策略。

#### 请求

```
PUT _ilm/policy/{policyID}
```

| 参数       | 描述     | 类型     | 是否必需 |
| ---------- | -------- | -------- | -------- |
| `policyID` | 策略 ID。 | `string` | 是       |

#### 查询参数

| 参数              | 描述                     | 类型     | 默认值 |
| ----------------- | ------------------------ | -------- | ------ |
| `if_seq_no`       | 仅当匹配此序列号时更新。   | `long`   | —      |
| `if_primary_term` | 仅当匹配此主分片任期时更新。| `long`   | —      |

#### 请求体

策略支持 ES 兼容的 **phases 格式**。系统内部会自动转换为 states 格式。

```json
PUT _ilm/policy/my_lifecycle
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "7d",
            "max_size": "50gb",
            "max_docs": 100000000
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          },
          "allocate": {
            "require": {
              "box_type": "warm"
            }
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "wait_for_snapshot": {
            "policy": "daily-backup"
          },
          "delete": {}
        }
      }
    }
  }
}
```

> **操作名称映射**：策略中使用的 ES 兼容名称会自动映射到内部操作名：
> - `set_priority` → `index_priority`
> - `allocate` → `allocation`
> - `forcemerge` → `force_merge`
> - `readonly` → `read_only`

#### 响应示例

```json
{
  "acknowledged": true
}
```

---

### 应用策略到索引模板

要让策略自动生效，需要在索引模板中指定策略名称和滚动索引的别名。

```json
PUT /_index_template/my_template
{
  "index_patterns": ["log-test-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "my_lifecycle",
      "index.lifecycle.rollover_alias": "log-test"
    }
  }
}
```

#### 创建初始写入索引

设置策略后，需要手动创建第一个托管索引并将其指定为写入索引。索引名称必须匹配模板模式且以数字结尾（后续 rollover 会自动递增）。

```json
PUT /log-test-000001
{
  "aliases": {
    "log-test": {
      "is_write_index": true
    }
  }
}
```

---

### 获取策略 {#get-policy}

通过 `policyID` 获取策略详情。

#### 请求

```
GET _ilm/policy/{policyID}
```

#### 响应示例

```json
{
  "_id": "my_lifecycle",
  "_version": 3,
  "_seq_no": 27,
  "_primary_term": 2,
  "policy": {
    "policy_id": "my_lifecycle",
    "description": "",
    "last_updated_time": 1682049301487,
    "schema_version": 17,
    "default_state": "hot",
    "states": [
      {
        "name": "hot",
        "actions": [
          {
            "retry": {
              "count": 3,
              "backoff": "exponential",
              "delay": "1m"
            },
            "rollover": {
              "min_size": "50gb",
              "min_doc_count": 100000000,
              "min_index_age": "7d"
            }
          },
          {
            "retry": {
              "count": 3,
              "backoff": "exponential",
              "delay": "1m"
            },
            "index_priority": {
              "priority": 100
            }
          }
        ],
        "transitions": [
          {
            "state_name": "warm",
            "conditions": {
              "min_index_age": "30d"
            }
          }
        ]
      },
      {
        "name": "warm",
        "actions": [
          {
            "retry": {
              "count": 3,
              "backoff": "exponential",
              "delay": "1m"
            },
            "force_merge": {
              "max_num_segments": 1
            }
          },
          {
            "retry": {
              "count": 3,
              "backoff": "exponential",
              "delay": "1m"
            },
            "allocation": {
              "require": { "box_type": "warm" },
              "include": {},
              "exclude": {},
              "wait_for": false
            }
          },
          {
            "retry": {
              "count": 3,
              "backoff": "exponential",
              "delay": "1m"
            },
            "index_priority": {
              "priority": 50
            }
          }
        ],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "90d"
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          {
            "retry": {
              "count": 3,
              "backoff": "exponential",
              "delay": "1m"
            },
            "wait_for_snapshot": {
              "policy": "daily-backup"
            }
          },
          {
            "retry": {
              "count": 3,
              "backoff": "exponential",
              "delay": "1m"
            },
            "delete": {}
          }
        ],
        "transitions": []
      }
    ],
    "ism_template": [
      {
        "index_patterns": [],
        "priority": 100,
        "last_updated_time": 1682049301487
      }
    ]
  }
}
```

> **注意**：创建策略时使用的是 ES 兼容的 phases 格式，但获取策略时返回的是内部 states 格式。操作名称也会转换为内部名称（如 `set_priority` 变为 `index_priority`）。

---

### 获取所有策略 {#list-policies}

不指定 `policyID` 即可列出所有策略。

#### 请求

```
GET _ilm/policy
```

#### 查询参数

| 参数          | 描述           | 类型     | 默认值                       |
| ------------- | -------------- | -------- | ---------------------------- |
| `size`        | 每页返回数量。   | `int`    | `20`                         |
| `from`        | 分页起始偏移量。 | `int`    | `0`                          |
| `sortField`   | 排序字段。      | `string` | `policy.policy_id.keyword`   |
| `sortOrder`   | 排序方向。      | `string` | `asc`                        |
| `queryString` | 搜索过滤条件。  | `string` | `*`                          |

#### 响应示例

```json
{
  "policies": [
    {
      "_id": "my_lifecycle",
      "_seq_no": 27,
      "_primary_term": 2,
      "policy": { "..." : "..." }
    }
  ],
  "total_policies": 1
}
```

---

### 更新策略 {#update-policy}

更新已有策略。必须在请求中指定 `if_seq_no` 和 `if_primary_term` 进行乐观并发控制。

#### 请求

```
PUT _ilm/policy/{policyID}?if_seq_no=27&if_primary_term=2
```

请求体格式与创建策略相同。

#### 请求示例

```json
PUT _ilm/policy/my_lifecycle?if_seq_no=27&if_primary_term=2
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "14d",
            "max_size": "100gb"
          }
        }
      },
      "delete": {
        "min_age": "180d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

### 删除策略 {#delete-policy}

通过 `policyID` 删除策略。

#### 请求

```
DELETE _ilm/policy/{policyID}
```

#### 请求示例

```
DELETE _ilm/policy/my_lifecycle
```

#### 响应示例

```json
{
  "_index": ".easysearch-ilm-config",
  "_type": "_doc",
  "_id": "my_lifecycle",
  "_version": 2,
  "result": "deleted",
  "forced_refresh": true,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 10,
  "_primary_term": 1
}
```

---

## 索引策略管理 API

### 将策略应用到索引 {#add-policy}

将指定策略应用到一个或多个索引。

#### 请求

```
POST /_ilm/add/{index}
```

| 参数    | 描述                               | 类型     | 是否必需 |
| ------- | ---------------------------------- | -------- | -------- |
| `index` | 索引名称，支持逗号分隔的多个索引。   | `string` | 是       |

#### 请求体

```json
{
  "policy_id": "my_lifecycle"
}
```

#### 响应示例

```json
{
  "updated_indices": 3,
  "failures": false,
  "failed_indices": []
}
```

---

### 移除索引策略 {#remove-policy}

从指定索引上移除生命周期策略。

#### 请求

```
POST /_ilm/remove/{index}
```

#### 请求示例

```
POST /_ilm/remove/log-test-000001
```

#### 响应示例

```json
{
  "updated_indices": 1,
  "failures": false,
  "failed_indices": []
}
```

---

### 变更索引策略 {#change-policy}

将索引当前的策略替换为新策略。

#### 请求

```
POST /_ilm/change_policy/{index}
```

#### 请求体

```json
{
  "policy_id": "new_lifecycle_policy",
  "state": "warm",
  "include": [
    { "state": "hot" }
  ],
  "is_safe": false
}
```

| 参数        | 描述                                      | 类型     | 是否必需 |
| ----------- | ----------------------------------------- | -------- | -------- |
| `policy_id` | 要切换到的新策略 ID。                       | `string` | 是       |
| `state`     | 切换后索引在新策略中的目标状态。             | `string` | 否       |
| `include`   | 按当前状态过滤要变更的索引。                 | `array`  | 否       |
| `is_safe`   | 是否为安全切换。                            | `boolean`| 否       |

#### 响应示例

```json
{
  "updated_indices": 1,
  "failures": false,
  "failed_indices": []
}
```

---

### 查看索引策略状态 {#explain}

获取索引的当前生命周期管理状态。支持使用索引模式匹配多个索引。

#### 请求

```
GET /_ilm/explain/{index}
```

#### 查询参数

| 参数               | 描述                    | 类型      | 默认值                 |
| ------------------ | ----------------------- | --------- | ---------------------- |
| `show_policy`      | 是否在响应中包含完整策略。 | `boolean` | `false`                |
| `validate_action`  | 是否包含操作验证结果。     | `boolean` | `false`                |
| `local`            | 是否使用本地集群状态。     | `boolean` | `false`                |
| `size`             | 每页返回数量。            | `int`     | `20`                   |
| `from`             | 分页起始偏移量。          | `int`     | `0`                    |
| `sortField`        | 排序字段。               | `string`  | `managed_index.index`  |
| `sortOrder`        | 排序方向。               | `string`  | `asc`                  |
| `queryString`      | 搜索过滤条件。            | `string`  | `*`                    |

#### 请求示例

```
GET /_ilm/explain/log-test*
```

#### 响应示例

```json
{
  "log-test-000003": {
    "index.lifecycle.name": "my_lifecycle",
    "index": "log-test-000003",
    "index_uuid": "58YtMBxaRcGBvOhHcJwQ5w",
    "policy_id": "my_lifecycle",
    "policy_seq_no": -2,
    "policy_primary_term": 0,
    "rolled_over": false,
    "index_creation_date": 1682060631245,
    "state": {
      "name": "hot",
      "start_time": 1682060706227
    },
    "action": {
      "name": "rollover",
      "start_time": 1682060764614,
      "index": 0,
      "failed": false,
      "consumed_retries": 0,
      "last_retry_time": 0
    },
    "step": {
      "name": "attempt_rollover",
      "start_time": 1682060764614,
      "step_status": "condition_not_met"
    },
    "retry_info": {
      "failed": false,
      "consumed_retries": 0
    },
    "info": {
      "message": "Pending rollover of index [index=log-test-000003]",
      "conditions": {
        "min_index_age": {
          "condition": "7d",
          "current": "2.2m",
          "creationDate": 1682060631245
        },
        "min_size": {
          "condition": "50gb",
          "current": "0b"
        },
        "min_doc_count": {
          "condition": 100000000,
          "current": 0
        }
      }
    },
    "enabled": true
  },
  "total_managed_indices": 1
}
```

---

### 重试失败操作 {#retry}

当生命周期管理操作执行失败时，使用此 API 重试。

#### 请求

```
POST /_ilm/retry/{index}
```

#### 请求体（可选）

```json
{
  "state": "hot"
}
```

指定 `state` 可以让索引从特定状态重新开始执行。

#### 响应示例

```json
{
  "updated_indices": 1,
  "failures": false,
  "failed_indices": []
}
```

---

## 状态转换条件

在策略的 states 格式中，每个状态可以定义转换条件。当条件满足时，索引自动进入下一个状态。

| 条件                | 描述                       | 类型     | 示例        |
| ------------------- | -------------------------- | -------- | ----------- |
| `min_index_age`     | 索引的最小年龄。             | `string` | `"30d"`     |
| `min_doc_count`     | 索引的最小文档数量（须 >0）。 | `long`   | `1000000`   |
| `min_size`          | 索引的最小大小（须 >0）。    | `string` | `"50gb"`    |
| `min_rollover_age`  | 自上次 rollover 以来的最小时间。| `string` | `"7d"`    |
| `cron`              | 基于 cron 表达式的时间条件。  | `object` | 见下方       |

---

## 操作参考

所有操作都支持以下通用可选参数：

| 参数              | 描述                     | 类型     | 默认值         |
| ----------------- | ------------------------ | -------- | -------------- |
| `timeout`         | 操作超时时间。             | `string` | 无             |
| `retry.count`     | 失败时的最大重试次数。      | `long`   | `3`            |
| `retry.backoff`   | 重试退避策略。             | `string` | `exponential`  |
| `retry.delay`     | 重试间隔时间。             | `string` | `1m`           |

> **操作名称兼容性**：在 phases 格式中，可以使用 ES 兼容名称（左列）；在 states 格式中，使用内部名称（右列）：
>
> | ES 兼容名称      | 内部名称         |
> | ---------------- | ---------------- |
> | `set_priority`   | `index_priority` |
> | `allocate`       | `allocation`     |
> | `forcemerge`     | `force_merge`    |
> | `readonly`       | `read_only`      |

---

### rollover

当现有索引满足指定的滚动条件时，将写入目标切换到新索引。

滚动目标可以是数据流或索引别名。当目标为数据流时，新索引将变为数据流的写索引，并且其代数将递增。

如果要滚动索引别名，别名及其写索引必须满足以下条件：

- 索引名称必须符合模式 `^.*-\d+$`，例如 `my-index-000001`。
- 必须将 `index.lifecycle.rollover_alias` 配置为要滚动的别名。
- 索引必须是别名的写索引。

| 参数                     | 描述                                            | 类型     | 是否必需 |
| ------------------------ | ----------------------------------------------- | -------- | -------- |
| `min_size` / `max_size`  | 当索引主分片总大小达到此值时触发滚动（不含副本）。  | `string` | 否       |
| `min_doc_count` / `max_docs` | 当索引文档数达到此值时触发滚动（不含副本和未刷新文档）。| `long` | 否   |
| `min_index_age` / `max_age`  | 当索引年龄达到此值时触发滚动。                    | `string` | 否       |
| `min_primary_shard_size` / `max_primary_shard_size` | 当最大主分片大小达到此值时触发滚动。| `string` | 否 |

> 每对参数中，左侧是内部名称，右侧是 ES 兼容别名。两种写法均可使用。

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50gb",
            "max_age": "7d",
            "max_docs": 100000000
          }
        }
      }
    }
  }
}
```

---

### set_priority / index_priority

设置索引的优先级，用于节点恢复后的索引恢复顺序。优先级高的索引先恢复。

| 参数       | 描述                       | 类型  | 是否必需 | 验证        |
| ---------- | -------------------------- | ----- | -------- | ----------- |
| `priority` | 索引优先级，值越大越优先。   | `int` | 是       | 必须 ≥ 0    |

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "actions": {
          "set_priority": {
            "priority": 50
          }
        }
      }
    }
  }
}
```

---

### forcemerge / force_merge

通过合并 Lucene 段（segment）来减少段的数量，优化查询性能。此操作在合并前会自动将索引设置为 `read_only` 状态。

**执行步骤**：设置只读 → 执行 force merge → 等待合并完成（共 3 步）。

| 参数               | 描述                       | 类型  | 是否必需 | 验证       |
| ------------------ | -------------------------- | ----- | -------- | ---------- |
| `max_num_segments` | 每个分片合并后的目标段数量。 | `int` | 是       | 必须 > 0   |

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      }
    }
  }
}
```

---

### readonly / read_only

将托管索引设置为只读，即 `index.blocks.write` 设为 `true`。

> **注意**：此设置不会阻止索引刷新（refresh）操作。

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "readonly": {}
        }
      }
    }
  }
}
```

---

### read_write

将托管索引设置为可读写状态，撤消 `read_only` 操作的效果。

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "read_write": {}
        }
      }
    }
  }
}
```

---

### replica_count

设置索引的副本分片数量。

| 参数                 | 描述                   | 类型  | 是否必需 | 验证       |
| -------------------- | ---------------------- | ----- | -------- | ---------- |
| `number_of_replicas` | 要设置的副本分片数量。   | `int` | 是       | 必须 ≥ 0   |

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "replica_count": {
            "number_of_replicas": 1
          }
        }
      }
    }
  }
}
```

---

### allocate / allocation

将索引分配到具有特定属性集的节点上。例如，将 `require` 设置为 `{ "box_type": "warm" }`，可以将索引数据迁移到 warm 类型节点。

| 参数       | 描述                                       | 类型     | 是否必需   | 默认值  |
| ---------- | ------------------------------------------ | -------- | ---------- | ------- |
| `require`  | 索引**必须**分配到具有所有指定属性的节点。    | `object` | 条件必需 ¹ | `{}`    |
| `include`  | 索引可以分配到具有**任一**指定属性的节点。    | `object` | 条件必需 ¹ | `{}`    |
| `exclude`  | 索引**不会**分配到具有**任一**指定属性的节点。 | `object` | 条件必需 ¹ | `{}`    |
| `wait_for` | 是否等待分配完成后再进入下一步操作。           | `boolean`| 否         | `false` |

> ¹ `require`、`include`、`exclude` 中至少有一个必须非空。

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "allocate": {
            "require": {
              "box_type": "warm"
            }
          }
        }
      },
      "cold": {
        "actions": {
          "allocate": {
            "include": {
              "box_type": "cold,frozen"
            },
            "wait_for": true
          }
        }
      }
    }
  }
}
```

---

### close

关闭托管索引。关闭的索引仍然存在于磁盘上，但不消耗 CPU 或内存资源，也不能被读取、写入或搜索。

如果需要长期保留数据但短期内无需搜索，且磁盘空间充足，关闭索引是一个好选择。需要时可以重新打开，比从快照恢复更简便。

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "cold": {
        "actions": {
          "close": {}
        }
      }
    }
  }
}
```

---

### open

打开一个已关闭的托管索引，恢复其读写和搜索能力。

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "open": {}
        }
      }
    }
  }
}
```

---

### delete

删除托管索引，将其从集群中完全移除。

| 参数              | 描述                                                   | 类型     | 是否必需 | 验证               |
| ----------------- | ------------------------------------------------------ | -------- | -------- | ------------------ |
| `timestamp_field` | 用于判断索引数据年龄的时间戳字段。须与 `min_data_age` 配对使用。| `string` | 否 ¹     | —                  |
| `min_data_age`    | 索引中最新数据的最小年龄要求。须与 `timestamp_field` 配对使用。 | `string` | 否 ¹     | 必须以天为单位（`d`），最小 `1d` |

> ¹ `timestamp_field` 和 `min_data_age` 必须同时指定或同时省略。

当两个参数都指定时，系统会：
1. 查询索引中按指定时间戳字段排序的最新文档。
2. 计算最新文档时间与当前时间的差值。
3. 仅当差值 ≥ `min_data_age` 时才执行删除。

这种机制可以防止意外删除仍在活跃使用的数据。

**基本用法**：

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

**带条件检查的删除**：

```json
PUT _ilm/policy/conditional_delete_policy
{
  "policy": {
    "phases": {
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {
            "timestamp_field": "@timestamp",
            "min_data_age": "90d"
          }
        }
      }
    }
  }
}
```

上面的示例中，即使索引满足了 30 天的 `min_age` 条件进入删除阶段，也只有当索引中最新的文档（基于 `@timestamp` 字段）已经至少 90 天未更新时，才会执行实际的删除操作。

---

### snapshot

将索引创建为快照备份。

**执行步骤**：执行快照 → 等待快照完成（共 2 步）。

| 参数         | 描述                               | 类型     | 是否必需 |
| ------------ | ---------------------------------- | -------- | -------- |
| `repository` | 已注册的快照仓库名称。               | `string` | 是       |
| `snapshot`   | 快照名称。支持 Mustache 模板变量。    | `string` | 是       |

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "cold": {
        "actions": {
          "snapshot": {
            "repository": "my_repository",
            "snapshot": "{{ctx.index}}"
          }
        }
      }
    }
  }
}
```

---

### shrink

将索引缩小到更少的主分片数量。

**执行步骤**：迁移分片 → 等待迁移完成 → 执行缩小 → 等待缩小完成（共 4 步）。

必须指定以下三个参数中的**恰好一个**来确定目标分片数：

| 参数                            | 描述                                      | 类型     | 验证                    |
| ------------------------------- | ----------------------------------------- | -------- | ----------------------- |
| `num_new_shards`                | 目标主分片数量。                            | `int`    | 必须 > 0                |
| `max_shard_size`                | 每个分片的最大大小，系统据此计算分片数。      | `string` | 必须 > 0                |
| `percentage_of_source_shards`   | 目标分片数为原分片数的百分比。               | `double` | 必须 > 0.0 且 < 1.0     |

| 可选参数                     | 描述                                                  | 类型     | 默认值 |
| ---------------------------- | ----------------------------------------------------- | -------- | ------ |
| `target_index_name_template` | 目标索引名称模板（Mustache 格式，语言须为 `mustache`）。 | `object` | 无     |
| `aliases`                    | 新索引的别名列表。                                      | `array`  | 无     |
| `force_unsafe`               | 是否允许不安全缩小。                                    | `boolean`| 无     |

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "shrink": {
            "num_new_shards": 1
          }
        }
      }
    }
  }
}
```

**按最大分片大小缩小**：

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "shrink": {
            "max_shard_size": "50gb"
          }
        }
      }
    }
  }
}
```

---

### alias

管理索引别名。可以在生命周期操作中添加或移除别名。

| 参数      | 描述                                    | 类型    | 是否必需 |
| --------- | --------------------------------------- | ------- | -------- |
| `actions` | 别名操作列表，每个操作为 ADD 或 REMOVE。  | `array` | 是       |

> **注意**：别名操作中不能指定 `index`/`indices` 参数，操作自动应用于当前托管索引。

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "alias": {
            "actions": [
              {
                "add": {
                  "alias": "my-data-archive"
                }
              },
              {
                "remove": {
                  "alias": "my-data-active"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

---

### rollup

在生命周期操作中自动创建 rollup 作业，将细粒度时序数据汇总为粗粒度数据以节省存储空间。

**执行步骤**：创建 Rollup 作业 → 等待 Rollup 完成（共 2 步）。

| 参数         | 描述                | 类型     | 是否必需 |
| ------------ | ------------------- | -------- | -------- |
| `ism_rollup` | ISM Rollup 配置对象。 | `object` | 是       |

#### ISM Rollup 配置

| 参数           | 描述                                                   | 类型     | 是否必需 | 验证                        |
| -------------- | ------------------------------------------------------ | -------- | -------- | --------------------------- |
| `description`  | Rollup 作业描述。                                       | `string` | 是       | —                           |
| `target_index` | 存储 rollup 结果的目标索引名称。                         | `string` | 是       | 不能为空                     |
| `page_size`    | 每批处理的文档数。                                       | `int`    | 是       | 1 ~ 10000                   |
| `dimensions`   | 维度定义列表。                                           | `array`  | 是       | 第一个必须是 `date_histogram` |
| `metrics`      | 指标聚合定义列表。                                       | `array`  | 是       | —                           |

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "cold": {
        "actions": {
          "rollup": {
            "ism_rollup": {
              "description": "Hourly rollup",
              "target_index": "rollup-my-index",
              "page_size": 1000,
              "dimensions": [
                {
                  "date_histogram": {
                    "source_field": "@timestamp",
                    "fixed_interval": "1h",
                    "timezone": "UTC"
                  }
                },
                {
                  "terms": {
                    "source_field": "host.name"
                  }
                }
              ],
              "metrics": [
                {
                  "source_field": "cpu.usage",
                  "metrics": [
                    { "avg": {} },
                    { "max": {} },
                    { "min": {} }
                  ]
                }
              ]
            }
          }
        }
      }
    }
  }
}
```

---

### wait_for_snapshot

在删除索引之前，等待指定的快照管理策略（SLM）完成执行。确保被删除索引的快照是可用的。

| 参数     | 描述                          | 类型     | 是否必需 |
| -------- | ----------------------------- | -------- | -------- |
| `policy` | 要等待的快照管理策略（SLM）名称。| `string` | 是       |

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "delete": {
        "actions": {
          "wait_for_snapshot": {
            "policy": "daily-backup"
          },
          "delete": {}
        }
      }
    }
  }
}
```

---

## 完整示例

以下示例展示一个典型的多阶段生命周期策略，涵盖从数据摄入到归档删除的完整流程：

```json
PUT _ilm/policy/production_lifecycle
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_size": "50gb",
            "max_age": "7d",
            "max_docs": 100000000
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          },
          "shrink": {
            "num_new_shards": 1
          },
          "allocate": {
            "require": {
              "box_type": "warm"
            }
          },
          "set_priority": {
            "priority": 50
          },
          "replica_count": {
            "number_of_replicas": 1
          }
        }
      },
      "cold": {
        "min_age": "90d",
        "actions": {
          "readonly": {},
          "allocate": {
            "require": {
              "box_type": "cold"
            }
          },
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "365d",
        "actions": {
          "wait_for_snapshot": {
            "policy": "nightly-backup"
          },
          "delete": {}
        }
      }
    }
  }
}
```

### 应用策略

```json
PUT /_index_template/production_logs
{
  "index_patterns": ["logs-prod-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "index.lifecycle.name": "production_lifecycle",
      "index.lifecycle.rollover_alias": "logs-prod"
    }
  }
}
```

```json
PUT /logs-prod-000001
{
  "aliases": {
    "logs-prod": {
      "is_write_index": true
    }
  }
}
```

### 检查生命周期状态

```
GET /_ilm/explain/logs-prod*?show_policy=true
```

### 手动为已有索引添加策略

```json
POST /_ilm/add/old-logs-2024
{
  "policy_id": "production_lifecycle"
}
```


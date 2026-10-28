---
title: "从 1.15.x 迁移到 3.0.0"
date: 0001-01-01
description: "从 1.15.x 升级到 2.5.1，完成系统索引迁移，再升级到 3.0.0 的操作指南。"
summary: "从 1.15.x 迁移到 3.0.0 #  本指南采用 1.15.x → 2.5.1 → 3.0.0，由 2.5.1 集中完成以下升级准备：
 节点兼容： 当前 3.0 不接受 1.15.x 节点加入，也不能直接读取其节点持久状态，须先经过兼容的 2.x 阶段。 旧 ZSTD 滚动写入修复： 1.15.x 使用 Lucene 8，2.5.x 使用仍支持写入这类旧索引的 Lucene 9。 2.5.0 的混部错误是误选了只读旧 codec；2.5.1 改为保留旧段读取、新段使用可写的 ZSTD V2。 3.0 使用 Lucene 10，常规写入兼容下限为 Lucene 9，对 Lucene 8 索引提供的是专用只读兼容，不能照搬“换 writer”的修复继续原地写入。 3.0 可以读取旧源并 reindex 到新索引，但这属于重建，不能替代第一跳滚动期间对旧索引的持续写入。 因此，要在滚动升级期间继续写入旧 ZSTD 索引，修复必须在参与混部的 2.x 包中交付。 系统索引迁移： 在 2.5.1 中把安全配置、ILM 配置和调度锁迁到可写的新实体，再进入 3.0。 否则旧系统索引在 3.0 中只能只读，配置更新和后台任务写入会受阻；当前 3.0 没有“先迁移、再启动系统服务”的引导流程。  ES 和 OpenSearch 都对大版本升级设置兼容边界，要求按受支持的版本路径升级，并提前处理不兼容索引等阻塞项。 例如，ES 升级到当前 9."
---


# 从 1.15.x 迁移到 3.0.0

本指南采用 **1.15.x → 2.5.1 → 3.0.0**，由 2.5.1 集中完成以下升级准备：

- **节点兼容：** 当前 3.0 不接受 1.15.x 节点加入，也不能直接读取其节点持久状态，须先经过兼容的 2.x 阶段。
- **旧 ZSTD 滚动写入修复：** 1.15.x 使用 Lucene 8，2.5.x 使用仍支持写入这类旧索引的 Lucene 9。
  2.5.0 的混部错误是误选了只读旧 codec；2.5.1 改为保留旧段读取、新段使用可写的 ZSTD V2。
  **3.0 使用 Lucene 10，常规写入兼容下限为 Lucene 9，对 Lucene 8 索引提供的是专用只读兼容，不能照搬“换 writer”的修复继续原地写入。**
  3.0 可以读取旧源并 reindex 到新索引，但这属于重建，不能替代第一跳滚动期间对旧索引的持续写入。
  因此，要在滚动升级期间继续写入旧 ZSTD 索引，修复必须在参与混部的 2.x 包中交付。
- **系统索引迁移：** 在 2.5.1 中把安全配置、ILM 配置和调度锁迁到可写的新实体，再进入 3.0。
  否则旧系统索引在 3.0 中只能只读，配置更新和后台任务写入会受阻；当前 3.0 没有“先迁移、再启动系统服务”的引导流程。

ES 和 OpenSearch 都对大版本升级设置兼容边界，要求按受支持的版本路径升级，并提前处理不兼容索引等阻塞项。
例如，[ES 升级到当前 9.x 前在 8.19.x 做升级准备](https://www.elastic.co/docs/deploy-manage/upgrade/deployment-or-cluster/elasticsearch)；
[OpenSearch 从 2.x 滚动升级到 3.x 须先到 2.19.x](https://docs.opensearch.org/latest/migrate-or-upgrade/rolling-upgrade/)，
OpenSearch 1.x 创建的不兼容索引需先在 2.x 中重建。2.5.1 借鉴的是这种“先准备、再升级”的方式，具体迁移机制由各产品实现。

2.5.1 是集中交付这些修复与迁移能力的版本选择。第一跳全量停机可避开 ZSTD 混部错误，但仍须完成节点状态与系统索引准备；
将迁移改放到 3.0 需要另行实现引导流程，不能直接省略这些步骤。

**无需先安装 2.5.0。** 已运行 2.5.0 的集群，升级到 2.5.1 后执行同样的迁移准备。
不能直接替换成 3.0.0 程序并启动，也不能让 1.15.x 与 3.0.0 节点混部。

2.5.1 延续 2.5.0 的历史索引与节点状态升级兼容逻辑，不要求存在 2.5.0 运行记录。
本文适用于至少有两个数据节点、业务索引配置主副本的多节点集群；部署前需用实际数据预演。

**2.5.1 提供升级到 3.0.0 前所需的系统索引迁移 API。** 全部节点必须先运行 2.5.1，再执行迁移。
普通 2.5.0 包不具备这套能力；迁移由服务端执行，无需额外迁移开关或外部工具。
本文操作须使用完整 2.5.1 发行包及匹配模块，后续 3.0.0 包也须支持读取迁移持久状态。

## 迁移路线与数据去向

推荐按下面的阶段执行，每个阶段验收后再进入下一阶段：

| 阶段 | 操作 | 进入下一阶段的条件 |
| --- | --- | --- |
| 1.15.x → 2.5.1 | 滚动升级或全量停机升级，所有角色直接更新到完整 2.5.1 包，保留原数据目录和安全配置 | 全部节点已到 2.5.1，主副本恢复，业务及安全配置可读写 |
| 系统索引迁移 | 在全 2.5.1 集群中通过 GET/POST 迁移仍需写入的旧系统索引 | GET 确认无需迁移；实际迁移的索引已完成实体与 alias 切换，认证、配置更新及后台任务正常 |
| 历史业务索引准备 | 选择重建为可写索引，或完成停写、写保护、flush 与副本检查 | 保留的历史索引具备完整持久化边界，备份可用 |
| 2.5.1 → 3.0.0 | 逐节点滚动升级或全量停机升级 | 全部节点为 3.0.0，业务、安全和后台功能验收通过 |

已经运行 2.5.0 的集群从 2.5.0 → 2.5.1 开始，无需退回 1.15.x，也无需重做已经完成的版本升级。

进入 3.0.0 后，不同索引的行为如下：

| 索引 | 处理方式 |
| --- | --- |
| 1.15.x 创建、符合兼容条件的 Lucene 8 普通或 ZSTD 业务索引 | 完成升级准备后只读查询；不能继续写入、force merge 或 clone/split/shrink |
| 在 2.5.x 新建的 Lucene 9 索引 | 可继续读写；需要同步验证分析器、插件与查询行为 |
| 旧 `.security`、ILM 配置、调度锁等运行期索引 | 在全 2.5.1 阶段迁到新建的可写实体，保留原逻辑名称 |
| 旧 k-NN 插件历史向量格式 | 不适用直接只读保留路径；需在能读取原索引的环境中准备数据，按目标字段映射重新导入并验证 |

索引是否需要迁移取决于创建版本、实际格式和用途。更新 `index.version.upgraded`、force merge 或原样快照恢复，
都不能代替新建索引并重新索引数据。进入 3.0.0 前，还需核对该版本的分词、ICU 排序键及 Java 客户端兼容性要求。

## 一、升级前盘点与备份

### 1. 准备操作入口

下文 `GET`、`PUT`、`POST` 示例是 REST 请求，可在 Easysearch UI 开发工具中逐条执行，或通过 HTTP 客户端发送。
每个代码块执行后先检查返回结果，满足该步骤的继续条件再执行下一步；不能把全篇请求一次性批量提交。

在终端使用用户名、密码调用迁移 API 的示例：

```bash
curl --cacert /path/to/ca.pem --user admin \
  'https://node.example:9200/_migration/system_features'
```

将地址、CA 路径和用户名替换为当前集群的实际值；上例会提示输入密码。POST 时加 `-X POST`，不传请求体。
安全开启时，涉及安全索引的手工读取、快照与恢复使用已有管理员证书；例如通过管理员证书查询安全索引 alias：

```bash
curl --cacert /path/to/ca.pem --cert /path/to/admin.pem --key /path/to/admin-key.pem \
  'https://node.example:9200/_alias/.security'
```

该 alias 查询用于迁移完成后，提前执行可能返回不存在。证书要求见[管理员证书配置]({{< relref "/docs/operations/security/configuration/tls.md" >}})。
下文 `upgrade_repo` 是已注册的快照仓库，`archive-logs` 是准备在 3.0.0 中只读保留的业务索引，均须替换为实际名称。

### 2. 记录升级前状态

记录集群和节点身份、索引设置、alias、模板及所用插件。下面的 REST 示例需要使用有相应权限的连接；
涉及 `.security` 的读取、复制和恢复时，使用已有的[管理员证书]({{< relref "/docs/operations/security/configuration/tls.md" >}})。

```http
GET /
GET /_cat/nodes?v&h=id,name,version,jdk,node.role,master&full_id=true
GET /_nodes/jvm,settings
GET /_cat/plugins?v
GET /_cat/indices?v&expand_wildcards=all
GET /_all/_settings?flat_settings=true&expand_wildcards=all
GET /_alias
```

重点检查：

- 业务索引是否使用 ZSTD、旧向量插件、自定义 codec、分词器或 ICU 排序键。
- 应用实际写入的索引或 write alias，以及升级期间会继续写入的后台模块。
- `.security` 或 `security.config_index_name` 指定的自定义安全索引，以及 ILM、调度锁、词典、模型等内部数据。
- 自定义 `security.system_indices.indices` 是否将模块索引限制为仅管理员证书可访问；内部角色执行的后台任务也受这类规则影响。
- 目标版本插件、词典、证书、keystore 和 JVM 参数是否齐全。

### 3. 创建并确认升级前备份

先按[快照仓库说明]({{< relref "/docs/features/data-retention/backup-restore.md" >}})注册仓库，再确认各相关节点可以访问：

```http
GET /_snapshot/upgrade_repo
POST /_snapshot/upgrade_repo/_verify
```

仓库查询和验证成功后，创建升级前快照：
下文快照名用于示例，每轮操作应使用唯一名称；已有同名快照时更换名称，不覆盖原备份。

```http
PUT /_snapshot/upgrade_repo/before-2-5?wait_for_completion=true
{
  "include_global_state": true,
  "partial": false
}

GET /_snapshot/upgrade_repo/before-2-5
```

确认快照为 `SUCCESS`、没有失败分片，并核对实际 `indices` 清单覆盖业务和所需内部索引。
关闭的索引需要单独处理，不能根据快照请求成功就认为它们已备份。安全相关快照和配置备份按敏感数据管理。
另行备份节点配置、证书、keystore、插件清单和外部词典，并在隔离环境验证恢复。

## 二、直接升级到全 2.5.1 集群

### 推荐：全量停机升级

1. 停止应用写入，暂停外部采集、同步和会修改数据的后台作业。
2. 对每个业务索引执行 `POST /<索引名>/_flush?force=true&wait_if_ongoing=true`，确认 `_shards.failed=0`，
   且 `_shards.successful=_shards.total`，再按原部署方式正常停止全部节点，确认原进程已经退出。
3. 在新安装目录准备完整的 2.5.1 发行包和配套插件，沿用原数据目录，无需安装 2.5.0。
4. 保留原 `cluster.name`、节点身份、`path.data`、发现配置、安全索引名称和 TLS 材料。
   以新包的 JVM 配置为基础迁入堆大小等自定义项，不能用旧 `jvm.options` 整体覆盖新配置。
5. 使用原服务账号和原部署方式启动 2.5.1：先启动具备 master 角色的节点，恢复选主，再启动其余节点。
   待节点全部加入后，执行下文“全 2.5.1 阶段验收”，通过后才恢复写入并进入系统索引迁移。

2.5.1 推荐使用 JDK 25；安全 agent 使用固定文件名 `easysearch-security-agent.jar`，
其余 JVM 参数要求见[2.5.1 升级说明]({{< relref "./_index.md#升级到-251系统索引迁移" >}})。

**已有集群升级不要重新运行 `bin/initialize.sh` 或 `bin/initialize-cluster.sh`。**
这两个入口用于新部署初始化，可能重新生成证书或更新初始配置；升级应保留已有安全数据和身份。

**更换安装目录时显式核对 `path.data`。** 如果旧部署使用安装目录内默认的 `data/`，新安装必须指向原数据目录的绝对路径，
否则可能启动成没有原节点身份和数据的新节点。保留原目录访问权限；不要在旧节点尚未退出时启动新进程访问同一目录。
服务管理器或容器也须同步程序目录、配置目录和 JDK，不能仍指向旧包。

### 如果选择第一跳滚动升级

2.5.1 修复了旧 ZSTD 索引在混部写入时误选只读格式、导致 `Old formats can't be used for writing` 的问题。
含旧 ZSTD 的 1.15.x 集群可直接滚动升级到 2.5.1；新节点读取原有旧段，并使用当前可写格式生成新段。
无需修改索引版本元数据，也无需先执行 force merge。

选择本路径时，不执行上面的“停止全部节点”。按以下顺序操作：

1. 从第一节记录中列出全部节点、角色、节点 ID、数据路径及升级顺序：先数据节点，再其它非专用 master 节点，最后专用 master。
   同时有 data/master 角色的节点按数据节点处理；每次只停止一个节点，保留选主所需的节点数。
2. 按原部署方式正常停止当前节点，确认进程退出；安装完整 2.5.1 包和配套插件，按上文要求迁入配置并指向原数据目录。
3. 使用原服务账号启动新节点，执行以下请求：

```http
GET /_cat/nodes?v&h=id,name,version,jdk,node.role,master&full_id=true
GET /_cluster/health?wait_for_status=yellow&wait_for_no_initializing_shards=true&wait_for_no_relocating_shards=true&timeout=60s
GET /_cat/shards?v
```

4. 确认该节点版本为 2.5.1、ID 与升级前一致、节点总数恢复；健康请求 `timed_out=false`、全部主分片可用，且分片恢复已完成。
   查询并写入实际业务索引，确认数据正确、写入无失败副本，然后才处理下一台节点。
5. 混部时如有未分配副本，先执行以下请求；仅因新旧节点版本限制无法把新版本分片复制到旧节点的临时 yellow 可继续。
   若主分片未分配、业务副本写入失败或出现真实恢复异常，停止后续升级，先查清原因。

```http
GET /_cluster/allocation/explain
```

6. 重复步骤 2～5，直到全部节点到达 2.5.1；再执行下文验收。全部主副本恢复为 green 后，才能启动系统索引迁移。

### 全 2.5.1 阶段验收

```http
GET /_cat/nodes?v&h=name,version,jdk,node.role,master
GET /_cluster/health?wait_for_status=green&wait_for_no_relocating_shards=true&timeout=60s
GET /_cat/shards?v
```

核对节点和数据目录与升级前一致；验证代表性业务的新增、更新、删除、查询和聚合。
**继续条件：** 节点数量与升级前一致，所有节点版本为 2.5.1，健康请求 `timed_out=false`、`status=green`，没有未完成的分片恢复。
安全开启的集群还应验证正确密码、错误密码拒绝、权限约束，以及通过安全配置 API 修改用户后各节点生效。
此时符合本路线的旧普通业务索引仍可读写，不需要因为未来将升级到 3.0.0 就提前永久停写。

## 三、在 2.5.1 中迁移系统索引

### 所有节点先到达 2.5.1

完成上一节后，直接使用当前全 2.5.1 集群执行迁移，无需再进行一轮版本升级。
若集群原本已运行 2.5.0，则先更新到 2.5.1，使用匹配的程序和模块，保留配置、数据与持久节点身份。
所有角色都必须更新，包括专用 master、协调节点和数据节点；更新期间不启动迁移。
保留自定义 JVM 配置时，须将原来带版本号的安全 agent 引用替换为 `lib/easysearch-security-agent.jar`，
详见[2.5.1 升级说明]({{< relref "./_index.md#升级到-251系统索引迁移" >}})。

迁移任务创建后，持久任务与结果会保留新的兼容边界，低于 2.5.1 的节点不能再加入；迁移结束后该限制仍保留。
不要把旧程序放回已写入迁移状态的数据目录。进入 3.0.0 前必须完成系统索引迁移及下文的业务准备。

### 1. 确认调用权限

使用集群管理身份访问任意正常集群入口，无需逐节点调用。沿用现有 `cluster_all` 权限，
或授予以下两项精确集群 action；只读监控或普通业务权限不足以调用：

```text
cluster:admin/migration/get_system_feature
cluster:admin/migration/post_system_feature
```

安全开启时沿用正常认证，不要求额外的管理员证书。迁移后台使用服务端受限内部身份，不保存调用者凭据。
安全索引的手工读取、快照备份与恢复仍遵守原有安全权限，不因可启动迁移就开放安全配置正文。

### 2. GET 检查系统索引迁移需求

查询哪些系统索引需要迁移、当前进度及失败原因，保存本次响应，作为迁移后核对索引名称和版本的依据：

```http
GET /_migration/system_features
```

响应按所属模块对系统索引分组：`features` 是模块分组列表，`indices` 是各组实际的系统索引；普通业务索引不在该接口范围内。
每组列出 `feature_name`、`minimum_index_version`、`migration_status` 与 `indices`；索引条目含 `index`、`version`，失败时增加 `failure_cause`。
最外层 `migration_status` 表示已登记模块的总体状态，不代表整个集群的所有旧索引均已处理。
当前登记的模块为 `security`、`index_management`、`job_scheduler`，自定义安全索引名称从 `security.config_index_name` 读取。

| 状态 | 含义与处理 |
| --- | --- |
| `NO_MIGRATION_NEEDED` | 已登记模块无需迁移；仍需自行核对其它插件与业务索引 |
| `MIGRATION_NEEDED` | 存在需要迁移的索引，完成备份后可提交 POST |
| `IN_PROGRESS` | 服务端持久迁移任务尚未完成，继续 GET 轮询 |
| `ERROR` | 检查 `failure_cause`、持久任务和 alias；重试期间可能仍显示上次失败，按步骤 7 区分 |

若总体状态已为 `NO_MIGRATION_NEEDED`，无需提交 POST，转到第四节，先处理接口未覆盖的内部写入，再准备历史业务索引。
无需迁移的系统索引可以保留原有实体名称，不要求统一转换为 alias。
若为 `IN_PROGRESS`，直接转到步骤 5 等待；若为 `ERROR`，转到步骤 7 处理，不能当作尚未开始直接提交。

### 3. 再次备份待迁移系统索引

总体状态为 `MIGRATION_NEEDED`，或已按步骤 7 确认失败清理完成、准备重试时，执行本步骤。
**POST 会删除完成迁移的旧物理源索引。** 提交前使用已有快照 API 创建并验证备份，覆盖实际系统索引，
并保存节点配置与证书。迁移 API 不接受 repository 参数，也不会代替运维创建备份。

```http
PUT /_snapshot/upgrade_repo/before-system-migration?wait_for_completion=true
{
  "include_global_state": true,
  "partial": false
}

GET /_snapshot/upgrade_repo/before-system-migration
```

**继续条件：** 快照为 `SUCCESS`，无失败分片，实际 `indices` 清单包含上一步 GET 返回的所有待迁移源索引。
备份未通过时，不执行迁移 POST。

### 4. 提交迁移

```http
POST /_migration/system_features
```

GET/POST 不接受请求体、feature 选择、源/目标或 protocol 参数。可用 `master_timeout` 限制联系 master 的等待时间，
它不限制整个迁移耗时。POST 接受后返回例如：

```json
{
  "accepted": true,
  "features": [
    {"feature_name": "index_management"},
    {"feature_name": "job_scheduler"},
    {"feature_name": "security"}
  ]
}
```

`accepted: true` 仅表示任务已接受。HTTP 连接断开不会取消后台任务；继续通过 GET 查进度。
POST 响应的 `features` 仅列出本次涉及的模块，未必包含示例中的全部项。
无候选或迁移预检查拒绝时返回 `accepted: false`，并给出 `reason` 或 `exception`；重复 POST 不会创建第二份任务。
权限不足、请求参数错误、集群写保护或联系 master 超时等，也可能直接返回 HTTP 错误，需同时检查 HTTP 状态与响应正文。
**继续条件：** 收到 `accepted: true` 后进入步骤 5；若为 `accepted: false`，先按 `reason` 或 `exception` 处理，
再 GET 确认当前状态，不把请求返回当作迁移完成。

服务端自动确认全部成员能力，暂停并排空已支持模块的后台工作；对每个待迁移索引，先创建 `<原名>-reindexed-for-3` 实体，
再建立源写屏障并执行 reindex，校验完整内容、版本、routing、物理格式和副本，全部通过后原子切换 alias 并恢复后台调度。
安全认证在迁移期间保持可用；源写屏障窗口内的安全配置更新会被拒绝，应待迁移完成后重试。

未知插件的可写旧内部索引、不支持排空的 provider、处于关闭状态或禁用 `_source` 的源索引、名称冲突等，需要由对应模块处理。
不能根据点号前缀统一删除、加只读或忽略。接口只盘点已登记的模块，不替代升级前的完整内部索引盘点。

### 5. 轮询，确认后台迁移完成

```http
GET /_migration/system_features
```

每隔数秒重复 GET：`IN_PROGRESS` 表示继续等待；`ERROR` 按步骤 7 核对持久任务，不能仅凭该状态判定本轮重试失败；
`NO_MIGRATION_NEEDED` 才进入结果验收。
若 POST 的结果未知或 HTTP 连接中断，也先 GET 查进度，不立即重复提交。迁移完成前不重启节点、不启动 3.0.0 节点。

### 6. 验证迁移结果

对步骤 2 中实际需要迁移的索引查询 alias。以下仅列出默认名称，自定义名称及其它登记索引以之前保存的 GET 响应为准：

```http
GET /_alias/.security
GET /_alias/.ilm-config
GET /_alias/.job-scheduler-lock
```

上述 alias 查询仅用于实际完成迁移的索引；自定义安全索引使用实际名称。无需迁移、仍保留实体名称的索引不适用 alias 查询。
总体状态须为 `NO_MIGRATION_NEEDED`，已迁移索引的 alias 须指向预期新实体且 `is_write_index=true`。

例如迁移 `.ilm-config` 后，`GET /_alias/.ilm-config` 应返回类似结构：

```json
{
  ".ilm-config-reindexed-for-3": {
    "aliases": {
      ".ilm-config": {"is_write_index": true}
    }
  }
}
```

再执行 `GET /_cluster/health?wait_for_status=green&wait_for_no_relocating_shards=true&timeout=60s`，
确认 `timed_out=false`、`status=green`，并完成以下功能检查：

- 各节点认证正常，错误密码被拒绝，普通用户不能直接读取或改写安全配置实体。
- 安全配置 API 的修改能持久化并在各节点生效。
- 实际启用的后台任务能够执行、更新运行状态并释放调度锁。
- 代表性节点重启后，迁移结果、认证、配置加载和后台任务保持正常。

源索引原有的 `index.blocks.write` 或 `index.blocks.read_only` 会使目标保留写保护。
迁移成功不等于解除用户设置的写保护；需要恢复写入时，确认原保护目的已解除后，再按实际目标索引检查并调整写保护。

### 7. 如果迁移失败

先保留 GET 的完整响应与相关节点日志，检查 `failure_cause` 和实际 alias 状态。
使用具备集群状态与任务查询权限的管理身份执行：

```http
GET /_cluster/state/metadata?filter_path=metadata.persistent_tasks.tasks
GET /_tasks?detailed=true
```

第一项检查是否仍有 `id` 为 `upgrade-system-indices` 的持久任务；不能仅因 `_tasks` 中没有运行任务就认定迁移已结束。
若持久任务仍在，`ERROR` 可能来自上次失败，也可能表示本轮正在失败清理；结合任务 `state` 与日志判断，继续观察，不重复 POST。
查询失败或权限不足也不能当作任务不存在。

迁移期间不重启节点、变更成员或直接修改源/目标。**alias 尚未切换时**，失败后服务端先确认子任务结束，
再清理本任务创建的目标和写屏障，保留用户原有写保护。
确认持久任务已结束、旧源仍在且本任务的目标和写屏障已清理后，重新执行步骤 3 的备份，再通过相同 POST 重试。
清理完成后，GET 仍可能保留上次失败的 `ERROR`，无需等待它先变成 `MIGRATION_NEEDED`；重试成功后会更新结果。
不要手工删除不明目标或解除仍在使用的屏障。
无法确认子任务结束时会保留任务与现场并报告 `ERROR`，需要先检查日志、现有 `_tasks` 与 alias，不能并发再开一轮。
alias 已切换后不能按切换前步骤回退；恢复应使用对应阶段的备份。快照恢复不会自动重演运行中的系统索引迁移任务。

## 四、准备历史业务索引

### 先处理迁移 API 未覆盖的内部写入

系统迁移 API 当前处理安全配置、ILM 配置和调度锁，不会自动迁移所有点号索引。
例如 ILM 作业还可能生成 `.ilm-managed-index-history-*`（或旧名称 `.easysearch-ilm-managed-index-history-*`）历史日志。
第一节盘点中发现的这些索引，也必须逐个确定是否仍有写入，不能因 GET 已为 `NO_MIGRATION_NEEDED` 就直接进入 3.0.0。

若 ILM 历史日志的 write alias 仍指向 1.15.x 创建的索引，在全 2.5.1 阶段执行以下步骤。
下面使用默认新名称，旧名称使用实际的 `.easysearch-ilm-managed-index-history-write`：

```http
GET /_alias/.ilm-managed-index-history-write
POST /.ilm-managed-index-history-write/_rollover
{}
GET /_alias/.ilm-managed-index-history-write
```

确认 rollover 响应 `rolled_over=true`、`acknowledged=true`、`shards_acknowledged=true`，
并记录 `old_index`、`new_index`。write alias 应已切到新索引；查询 `GET /<new_index>/_settings?flat_settings=true`，
其中 `<new_index>` 替换为响应中的实际新索引名，`index.version.created` 应为 `2050199`（对应 2.5.1）。
对 `old_index` 以及其它仍保留的旧历史日志实体，按下文“在 3.0.0 中只读保留的业务”执行写保护、flush 和主副本检查。
不要对 write alias 加写保护；它需要继续向新索引写入。源索引已预先写保护导致 rollover 失败时，先查清原保护原因，不能直接解除。

其它模块的旧内部索引也按相同原则盘点：仍需写入的按所属模块支持的方式重建或切换写入口，
只读保留的完成写保护与 flush；没有经过验证的处理方式时，先停止后续大版本升级。

### 需要继续写入的业务

在全 2.5.1 阶段完成以下步骤。下面以源 `orders-v1`、新目标 `orders-v2` 为例，必须替换为实际业务索引；不要用于系统索引。

1. 停止应用、采集及后台作业对源的写入，并建立写屏障：

```http
PUT /orders-v1/_block/write
POST /orders-v1/_refresh
GET /orders-v1/_mapping
GET /orders-v1/_settings?flat_settings=true
```

2. 根据实际业务 mapping、分析器、codec、分片和副本数创建 `orders-v2`；目标必须是本次新建的空索引。
   创建方法见[重建索引]({{< relref "/docs/operations/data-management/reindex.md" >}})。
   不直接提交旧 `_settings` 响应；索引版本、UUID、历史写保护等不应原样复制，目标须允许写入，reindex 本身也不会复制索引配置。
3. 执行复制：

```http
POST /_reindex?wait_for_completion=true&refresh=true
{
  "source": {"index": "orders-v1"},
  "dest": {"index": "orders-v2"}
}
```

若连接中断，先通过 `GET /_tasks?actions=*reindex*&detailed=true` 核查原任务与目标数据，不直接向已有数据的目标重复提交。

4. 确认 `timed_out=false`、`failures=[]`、`version_conflicts=0`，空目标的 `created=total`；再核对数据：

```http
GET /orders-v1/_count
GET /orders-v2/_count
```

5. 除文档数相等外，核对完整 ID/内容、代表性查询和聚合，确认目标主副本均可用。
   数据不同或复制失败时不切换入口，保留源数据并排查。
6. 若应用使用 write alias，按[alias 切换]({{< relref "/docs/operations/data-management/aliases.md" >}})将其切到 `orders-v2`；
   若应用直接使用实体索引名称，先将读写配置更新到目标。确认应用已切换后恢复写入，并验证新增、更新、删除与查询。
   源索引保持停写，完成业务验收与备份前不删除。

如果旧源索引仍保留在集群中进入 3.0.0，也必须按下一节完成写保护和 flush，不能仅因业务入口已经切走就跳过准备。

以上采用停写复制。需要不停写迁移时，须另行设计并验证增量同步，避免遗漏复制开始后的更新和删除。

也可先把符合条件的历史索引只读保留，在 3.0.0 中重建到新索引后再切换。
若还涉及分词或 ICU 排序键变化，应按 3.0.0 的目标分析链重新生成并验证数据，不能仅以物理格式升级作为验收。
具体 API 见[重建索引]({{< relref "/docs/operations/data-management/reindex.md" >}})与
[alias 切换]({{< relref "/docs/operations/data-management/aliases.md" >}})。

### 在 3.0.0 中只读保留的业务

在第一台 3.0.0 节点启动前，对每个计划保留的 Lucene 8 历史业务索引完成下面的准备。
示例中的 `archive-logs` 必须替换为明确盘点过的业务索引，不能对 `*` 或所有系统索引统一执行。

对每个需要只读保留的业务索引，依次执行以下步骤；多个索引逐一处理：

1. 停止应用、采集和后台作业对该索引的所有写入。
2. 建立写屏障，确认响应无失败，再 flush，检查全部主副本的持久状态：

```http
PUT /archive-logs/_block/write

POST /archive-logs/_flush?force=true&wait_if_ongoing=true

GET /archive-logs/_settings?flat_settings=true

GET /archive-logs/_stats?level=shards

GET /archive-logs/_segments
```

3. 确认 `index.blocks.write=true`，flush 的 `_shards.failed=0`、`successful=total`，且覆盖预期的全部主副本。
   检查每个分片的 `max_seq_no`、`local_checkpoint` 和 `global_checkpoint` 对齐，并核对主副本查询返回的完整业务数据；
   不能只看文档数或集群 green。
**写保护和 flush 必须都完成**：写屏障阻止后续写入，但不会自动把此前操作变成完整的持久化 commit。

4. 保持这些历史索引的写屏障，进入 3.0.0 后也不解除；需要继续写入的业务应完成上一节的新建与 reindex。

系统索引迁移和历史业务准备完成后，再创建一份进入 3.0.0 前的快照：

```http
PUT /_snapshot/upgrade_repo/before-3-0?wait_for_completion=true
{
  "include_global_state": true,
  "partial": false
}

GET /_snapshot/upgrade_repo/before-3-0
```

确认快照为 `SUCCESS`、无失败分片，并覆盖迁移后的系统实体与准备好的业务索引。
备份原来的 `cluster.routing.allocation.enable` 等运维设置，便于恢复。

## 五、从 2.5.1 升级到 3.0.0

此时应同时满足：所有节点运行 2.5.1；需迁移的系统索引已完成迁移，活动系统写入口可正常写入；历史业务已完成上述准备；
备份可用；新包、插件及 JVM 配置齐全。

### 滚动升级

1. 每次只停止一个节点，保留选主和业务可用性。先升级数据节点及其它非专用 master 节点，最后升级专用 master。
   同时有数据和 master 角色的节点按数据节点处理，一台节点只升级一次。
2. 使用完整 3.0.0 包及匹配插件，保留原数据目录、节点身份与 TLS 配置，按新包同步 JVM 参数。
3. 启动该节点，等待它以原节点身份加入，执行以下检查。

```http
GET /_cat/nodes?v&h=id,name,version,jdk,node.role,master&full_id=true
GET /_cluster/health?wait_for_status=yellow&wait_for_no_initializing_shards=true&wait_for_no_relocating_shards=true&timeout=60s
GET /_cat/shards?v
```

确认当前节点为 3.0.0、节点 ID 与升级前一致、节点总数恢复，健康请求 `timed_out=false`。
安全配置加载完成后核对认证、历史业务查询与新业务读写，再处理下一台。混部的副本版本限制按第二节方法解释；真实恢复失败时停止后续升级。

4. 如临时调整了分片分配设置，按升级前记录的值恢复；全部节点到 3.0.0 后，确认分片恢复完成并达到 green。

### 全量停机升级

1. 完成同样的系统与业务索引准备、进入 3.0.0 前的备份，并停止应用写入及其它写入来源。
2. 按原部署方式正常停止全部节点，确认原进程退出。
3. 使用完整 3.0.0 包和配套插件，沿用原数据目录、节点身份和安全配置，按新包同步 JVM 参数。
4. 使用原服务账号先启动具备 master 角色的节点，恢复选主，再启动其它节点；通过下文最终验收后恢复可写业务的写入。

全量停机不会替代系统索引迁移、历史索引写保护或 flush。

### 最终验收

```http
GET /_cat/nodes?v&h=name,version,jdk,node.role,master
GET /_cluster/health?wait_for_status=green&wait_for_no_relocating_shards=true&timeout=60s
GET /_cat/shards?v
```

除版本和分片状态外，还需检查：

- 历史只读索引的文档、关键查询和聚合与升级前一致；写请求被拒绝。
- 新建的可写业务索引支持新增、更新和删除，主副本查询结果一致。
- 系统 alias 指向正确实体，安全配置修改、实际后台任务和锁释放正常。
- 普通用户的索引、文档及字段权限没有被放宽。
- 代表性节点重启后配置和数据可恢复，客户端、分词、语义搜索及实际使用的插件功能正常。

## 常见阻塞与回退

| 现象 | 处理方式 |
| --- | --- |
| 1.15.x 直接启动 3.0.0 时报旧格式，或节点不能加入 | 恢复到该阶段升级前的环境与备份，按 2.5.1 中间阶段重新规划；不要删除节点状态文件强行启动 |
| 1.15.x 与 2.5.x 混部写旧 ZSTD 时出现 `Old formats can't be used for writing` | 2.5.1 已修复该问题；检查目标节点是否使用完整 2.5.1 包及配套模块。仍失败时停止继续滚动并排查，不修改旧 codec 的只读限制 |
| `historical index requires a write or read_only block` | 若是业务索引，回到升级准备检查写保护；若是活动系统索引，先完成物理迁移，不能只加 block |
| `historical recovery checkpoint ... != complete commit boundary ...` | 检查停写、写屏障及全部副本 flush 是否完成；保留源数据并查清恢复状态，不删除 translog 或放宽校验 |
| 迁移 API 不存在，或 POST 返回版本、成员能力、provider 错误 | 核实所有角色的节点已更新到 2.5.1 及匹配模块，不跳过服务端检查 |
| 认证在节点刚启动时返回 503 | 等待配置加载；持续 503 应查安全索引分配和加载日志。错误密码 401 需单独排查 |
| alias 已切换，但 ILM 等任务不执行 | 检查实际模块任务、调度状态及自定义系统索引保护规则，确认内部角色能按原权限执行 |

升级后不支持将旧版本程序直接放回已升级的数据目录，也不支持让旧版本节点重新加入完成升级的集群。
需要回退时，应重新部署原版本，并从该阶段升级前已验证可恢复的快照及配置恢复。
3.0.0 上产生的新写入需另行纳入回退的数据处理方案。


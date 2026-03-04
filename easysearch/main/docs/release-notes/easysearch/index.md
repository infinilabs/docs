---
title: "Easysearch"
date: 0001-01-01
summary: "版本发布日志 #  这里是 INFINI Easysearch 历史版本发布的相关说明。
Latest (In development) #  Breaking changes #  Features #  Bug fix #  Improvements #  2.1.0 (2026-2-13) #  Breaking changes #  Features #   新增 Rules 规则引擎插件，提供高性能的规则匹配能力  支持 linux-x64 和 linux-aarch64 架构 支持 Ingest Pipeline 集成，数据写入时自动匹配规则并添加标签 支持复杂的规则表达式（AND/OR/NOT、near、正则、数值范围等） 支持百万级规则库，匹配性能是传统方案的上百倍。 支持多节点集群自动同步和广播编译规则 节点启动时自动同步缺失的规则库 规则库同步期间自动保护写入，确保规则完整性 本地元数据文件持久化记录编译历史，支持规则库文件丢失后的自动恢复   新增形态学分析插件（analysis-morphology），支持俄语和英语的形态分析  精准还原：基于词典将动词时态、名词格位等还原为标准原型（如 went → go） 词元扩展：同时索引原词与关联词根（如runner → runner, run），实现智能搜索匹配 高召回率：解决俄语复杂的变格与变位搜索难题，确保不同语法形式下均能精准检索   审计日志新增动态指定用户进行审计的功能 UI 插件新增如下能力  支持审计日志在线查看 支持审计日志模块动态配置 新增数据探索页面    Bug fix #  Improvements #   将“结巴”分词插件日志迁移至 Log4J，并降低周期性任务的日志级别以减少冗余  2."
---


# 版本发布日志

这里是 INFINI Easysearch 历史版本发布的相关说明。

## Latest (In development)
### Breaking changes
### Features
### Bug fix
### Improvements

## 2.1.0 (2026-2-13)

### Breaking changes

### Features

- 新增 Rules 规则引擎插件，提供高性能的规则匹配能力
  - 支持 linux-x64 和 linux-aarch64 架构
  - 支持 Ingest Pipeline 集成，数据写入时自动匹配规则并添加标签
  - 支持复杂的规则表达式（AND/OR/NOT、near、正则、数值范围等）
  - 支持百万级规则库，匹配性能是传统方案的上百倍。
  - 支持多节点集群自动同步和广播编译规则
  - 节点启动时自动同步缺失的规则库
  - 规则库同步期间自动保护写入，确保规则完整性
  - 本地元数据文件持久化记录编译历史，支持规则库文件丢失后的自动恢复
- 新增形态学分析插件（analysis-morphology），支持俄语和英语的形态分析
  - 精准还原：基于词典将动词时态、名词格位等还原为标准原型（如 went → go）
  - 词元扩展：同时索引原词与关联词根（如runner → runner, run），实现智能搜索匹配
  - 高召回率：解决俄语复杂的变格与变位搜索难题，确保不同语法形式下均能精准检索
- 审计日志新增动态指定用户进行审计的功能
- UI 插件新增如下能力
  - 支持审计日志在线查看
  - 支持审计日志模块动态配置
  - 新增数据探索页面
  
### Bug fix

### Improvements
- 将“结巴”分词插件日志迁移至 Log4J，并降低周期性任务的日志级别以减少冗余

## 2.0.2 (2025-12-17)

### Breaking changes

### Features
- 语义搜索新增支持 NestedQueryBuilder
- KNN mapping 的 L 和 k 参数支持大小写不敏感，提升易用性

### Bug fix
- 修复了开发者工具的主题颜色显示问题

### Improvements
- UI 插件静态文件支持 gzip 压缩，加快页面加载
- 优化了图标资源大小
- 调整了内部构建流程和 CSP 策略


## 2.0.1 (2025-12-03)

### Breaking changes

### Features

### Bug fix
- 修复 UI 模块一些显示问题

### Improvements


## 2.0.0 (2025-11-21)

### Breaking changes
- Easysearch 2.0.0 版本底层 Lucene 更新到 9.12.2
- 新增 ui 插件，为 Easysearch 提供了轻量级界面化管理功能，不再依赖第三方对集群进行管理，真正做到开箱即用

### Features
- 兼容 1.15.x 版本的索引，可无缝升级
- 新增 UI 插件，涵盖从集群，节点，索引，到分片等不同维度的监控和管理功能
- 支持关闭 security 进入 UI

### Bug fix

### Improvements
- range 查询，按数字类型字段排序，相比旧版本效率大幅提升

## 1.15.6 (2025-10-31)
### Breaking changes
### Features
### Bug fix
- 修复 bug
### Improvements


## 1.15.5 (2025-10-20)
### Breaking changes
### Features
### Bug fix
- 修复索引不存在时没有正确返回客户端 index_not_found_exception 的错误
### Improvements


## 1.15.4 (2025-10-11)
### Breaking changes
### Features
### Bug fix
- 修复 客户端请求的路径为 `/ui` 时不能正常显示欢迎页的问题
### Improvements
- 改进 rollup 索引的模版
- 增强对 cloud 的集成

## 1.15.3 (2025-09-28)
### Breaking changes
### Features
- 为索引操作添加新的菜单项
- 节点详情新增树状统计字段存储
- 添加热点线程页面
### Bug fix
### Improvements


## 1.15.2 (2025-09-21)

### Breaking changes

### Features
- 集群页面新增「临时配置」、「限流限速」以及「其他配置」页面
- 索引详情页面新增「限流」页面

### Bug fix

### Improvements
- 重构索引详情页面的「设置」页面
- 去掉 ILM 配置索引的前缀，并兼容旧索引
- index-management 从plugin 移动到 modules
- 精简证书错误时的日志输出


## 1.15.1 (2025-09-14)

### Breaking changes
-  更新创建搜索管道的 API 的 json 结构和说明文档

### Features
- ui 插件新增
  - 备份快照管理功能
  - 跨集群复制管理功能
  - 数据流管理功能

### Bug fix

### Improvements
- 改进 search_pipeline 的统计指标

## 1.15.0 (2025-09-08)

### Breaking changes
- 新增 ui 插件，为 Easysearch 提供了轻量级界面化管理功能，不再依赖第三方对集群进行管理，真正做到开箱即用
- 正式发布混合搜索模块，结合了关键词搜索和语义搜索，以提升搜索相关性
- 针对安全模块的角色名称进行规范，废弃不符合规范的角色，

### Features

- 新增 ui 插件，涵盖从集群，节点，索引，到分片等不同维度的监控和管理功能。
- ai 插件正式提供`混合搜索`能力
- 引入 license
- 允许动态的跨模板重用设置

### Bug fix

### Improvements
- 改进角色名称和描述
- 增加 数据流（Data streams）说明文档
- 更新搜索管道相关文档


## 1.14.0 (2025-07-25)

### Breaking changes
- AI 模块 从 modules 迁移至 plugins 目录下，方便调用 knn 插件
- 旧的文本向量化接口 `_ai/embed` 已不再支持，将在后续版本删除

### Features

- 插件模块新增完整的文本嵌入模型集成功能，涵盖从数据导入到向量检索的全流程
- 新增语义检索 API，简化向量搜索使用流程
- 新增语义检索处理器配置大模型信息
- 新增搜索管道（Search pipelines），轻松地在 Easysearch 内部处理查询请求和查询结果
- 多模型集成支持
  - OpenAI 向量模型：直接调用 OpenAI 的嵌入接口（如 text-embedding-3-small）
  - Ollama 本地模型：支持离线环境或私有化部署的向量生成
- ik 分词器提供 reload API，能够对存量自定义词典进行完整更新
- ik 分词器能够通过词库索引对默认词库进行自定义添加

### Bug fix

### Improvements
- 增强数据摄取管道（ingest pipeline）
  - 在数据索引阶段支持文本向量化，文档可自动生成向量表示
  - 导入数据时通过 ingest 管道进行向量化时支持单条和批量模式，适配大模型的请求限制场景
- 更新 Easysearch Docker 初始化文档
- ik 分词器优化自定义词库加载逻辑，减少内存占用


## 1.13.1 (2025-06-29)

### Breaking changes

### Features

- 插件模块新增 dependencyModule 配置项，用于声明共享的 common 模块依赖

### Bug fix

- 修复了前缀查询请求在只包含一个字符时的空指针错误

### Improvements

- 精简了部署后的 modules和 plugins 大小


## 1.13.0 (2025-06-16)

### Breaking changes

### Features

- Rollup 新增对已创建 job 的 interval 和 page_size 参数更新的 api
- Rollup 索引数据增加 unique 字段标识当前 job 的数据
- Rollup 配置增加 window_start_time 字段，当重建 Rollup 时 会把历史 metadata 的最新时间戳写到 window_start_time 里

### Bug fix

### Improvements

- Rollup 会自动检测源数据索引是否有新增字段，并更新到 metrics
- ILM 删除索引时 增加对 Rollup 的运行状态判断，未处理完的索引不会删除


## 1.12.3 (2025-05-31)

### Breaking changes

### Features

- Rollup Job 支持断点续跑： 当检测到历史 Rollup 索引及元数据时，新创建的 Rollup Job 会自动从中断的状态点继续处理，避免重复计算，显著节省计算资源

### Bug fix

- 修复前端 Rollup 查询时如果只有历史数据，不能正确返回 pipeline 聚合结果的错误

### Improvements

- 优化 Rollup job 的 top_hits 聚合效率
- 提升启用 source_reuse 时的文档获取性能


## 1.12.2 (2025-05-18)

### Breaking changes

### Features

-  NodeStats 新增 rollup 的 运行状态，方便监控
-  更新 ik-analyzer 文档

### Bug fix

### Improvements

- 当设置了 JAVA_HOME 时，优先使用本地 JDK
- 新增安装为 linux 服务的文档


## 1.12.1 (2025-04-28)

### Breaking changes

### Features

- 索引合并新增按照日期范围合并的策略：通过 index.merge.policy.time_range_field 配置项指定合并依据的字段

### Bug fix

- 修复 ollama_url 不能动态更新的错误
- 修复 ollama api 未正确兼容单个文本请求
- 索引生命周期管理 delete action 按文档最新时间删除时修正为按降序排序

### Improvements


## 1.12.0 (2025-03-28)

### Breaking changes

### Features

- Rollup 新增 write_optimization 配置项，启用后采用自动生成文档 ID 的策略，大幅提升写入速度
- Rollup 现在支持针对 job 级别配置 rollover 的 max docs


### Bug fix

- Rollup 修复带有内嵌的 pipeline 聚合时不能和原始索引聚合正常合并的问题


### Improvements

- 优化了 rollup 索引字段名长度，减小 rollup job 运行时的内存占用


## 1.11.1 (2025-03-14)

### Breaking changes

### Features

- 新增 AI 模块，集成 Ollama embedding API，支持文本向量化


### Bug fix

- 修复 DateRange 聚合在 Rollup 查询中无法正确合并的问题


### Improvements

针对用户使用体验进行了多项改进，包括：
- 弃用 KNN 模块中的 `index.knn` 配置项，（此配置项和其他功能经常发生冲突） 简化配置逻辑，该配置项将在后续版本中移除
- 将 KNN 搜索功能从插件形式集成为内置功能，无需额外安装即可使用
- 将跨集群复制（CCR）功能从插件形式集成为内置功能，开箱即用
- 优化索引配置更新验证：增加非动态配置项的值比对，避免误报

## 1.11.0 (2025-02-28)


### Features

- 新增 wildcard 数据类型
- 新增 Point in time 搜索 API
- 新增异步搜索 API
- 数值字段添加 doc-values 搜索支持
- 日期字段添加 doc-values 搜索支持
- 新增 IK 分词器自定义词典使用文档

### Bug fix

### Improvements

- 优化 Lucene flush 的 segment 大小，减少 I/O 开销

## 1.10.2 (2025-02-17)

### Features

- lucene 版本更新
    - lucene 版本更新到 8.11.4，是 lucene8.x 系列的最后一个版本
    - jna 更新到 5.12.1

- IK 分词器: 增强词典配置的灵活性和可扩展性
    - 支持字段级别的词典配置，用户可通过自定义 tokenizer 为不同索引、不同字段配置专用词典
    - 优化词典管理机制
    - 支持自定义词典与 IK 默认词典合并使用
    - 词库数据存储在可配置的索引中，支持实时更新
    - 可使用内置词库索引或自定义词库索引(需保持相同结构)

- 索引生命周期管理
    - delete action 支持同时基于索引创建时间和文档最新时间戳来执行删除操作

### Bug fix

- Rollup
    - 修复了平均值(avg)聚合计算错误

### Improvements

- 优化 rollup 索引的创建流程

## 1.10.1 (2025-01-24)

### Features

- Rollup 增加支持聚合的种类
    - 增加支持 Filter aggregation，某些场景可以用来替代 query
    - 增加针对个别字段自定义 special_metrics 指标的配置项
    - 增加支持 Bucket sort aggregation
- Rollup 查询 API 提供了 debug 参数，有助于调试

### Bug fix

- 修复数据节点和 master 节点角色分离时，Security 索引创建失败问题

### Improvements

- Rollup 查询 增加 response 标识是否有 rollup 数据
- Rollup response total hits 不再为 0
- Rollup job 支持 更新操作，通过更新索引文档实现
- rollup.hours_before 配置项只影响查询时间范围，不影响写入


## 1.10.0 (2025-01-11)

### Features

- Rollup 功能增强：新增并发限制、任务失败自动重启功能，支持批量启动、停止 Job， 并支持 date_range 聚合。
- 字段类型功能优化：新增 flattened_text 和 match_only_text 字段类型，支持更多查询场景；

## 1.9.0 (2024-10-17)

### Breaking changes

### Features

- 发布 rollup 功能
- 支持自动对 rollup 索引进行滚动，无需外部触发
- 支持 avg sum max min value_count percentiles 指标类型的聚合
- 支持 terms 聚合
- 支持对指标聚合进行 Pipeline 聚合
- 支持聚合前先对数据进行过滤
- 进行聚合查询时支持直接搜索原始索引，不用更改搜索代码

- 增加适配 logstash8.x 的请求 header
- \_cat/templates 增加 lifecycle 和 rollover 列的展示

### Bug fix

- 修复 rest-api template 测试错误

## 1.8.3 (2024-08-13)

### Bug fix

- 修复特定场景下 lucene 栈溢出问题
- 修复特定场景下字节处理空指针问题

### Improvements

- 更新依赖库至安全版本
- 优化开启 source_reuse 后内存使用性能
- 增加初始化密码环境变量,可手工设置 [EASYSEARCH_INITIAL_ADMIN_PASSWORD]({{< relref "/docs/deployment/install-guide/docker.md" >}}) 环境变量。

## 1.8.2 (2024-06-06)

### Breaking changes

### Features

### Bug fix

- 修复 source_reuse 与 索引 mapping 的 enable: false 冲突

### Improvements

- 升级部分依赖包版本，commons-collections to 3.2.2, snakeyaml to 2.0
- 优化 CCR 同步性能及调整 CCR 全局配置参数
- 优化插件配置命名，去除"plugins."
- 优化配置文件目录获取命名

## 1.8.0 (2024-04-30)

### Breaking changes

### Features

- 增加写入限流功能，可针对节点级（数据节点和协调节点）、分片级

### Bug fix

- 修复查询数据结果为空时，聚合出错问题
- 修复 Bundle 包在 MacOS 环境下 JDK 路径出错问题

### Improvements

## 1.7.1 (2024-03-01)

### Breaking changes

### Features

### Bug fix

- 修复 \_meta 不为空且 启用 source_reuse 时的映射解析错误
- 修复 source_reuse 下对多值还原不正确的问题
- 修复 source_reuse 和 alias 类型字段的冲突

### Improvements

- 改进跨集群复制的数据加载，增加对 source_reuse 索引的支持
- 内存断路器在触发 GC 时增加 full GC 回退
- 针对 String 类型的 TermsAgg 增加分片级别的内存断路检测

## 1.7.0 (2023-12-15)

### Breaking changes

### Features

- 发布快照搜索功能 Beta 版本，此功能旨在提高对已备份数据的使用效率。让用户利用对象存储（如 AWS S3、MinIO、Microsoft Azure Storage、Google Cloud Storage 等）技术来大幅降低存储成本。

### Bug fix

- 修复单个节点场景下，从快照恢复多个 shard 的索引时，恢复不完整的问题
- 修复无法删除索引已关联的 ILM 策略问题

### Improvements

- 初始化脚本优化，新增重复执行判断

## 1.6.2 (2023-12-01)

### Breaking changes

### Features

### Bug fix

- 修复跨集群复制（CCR）不能对自动滚动生成的索引进行同步的问题

### Improvements

- 优化初始化脚本，增加-s/-slient 自动安装参数。
- 新增含 jdk/plugins 的 [bundle 安装包](https://release.infinilabs.com/easysearch/stable/bundle/)

## 1.6.1 (2023-10-19)

### Breaking changes

### Features

- 增加 analysis-icu 插件

### Bug fix

- 修复 JDK 17 及更高版本运行告警及异常

### Improvements

- 安装脚本优化，避免脚本使用不当出现错误
- source_reuse 增加对 float，double，geo_point，half_float，ip 类型字段的压缩
- 优化启用 source_reuse 时的写入速度，压缩的字段越多，写入速度越快

## 1.6.0 (2023-09-22)

### Breaking changes

### Features

- 增加 \_field_usage_stats api，统计索引每个字段的访问次数
- 新增 \_disk_usage api，可以分析指定索引每个字段的磁盘占用大小
- 增加 flattened 类型，将 json 对象作为字符串处理，可以减少嵌套 json 型的文档的大小

### Bug fix

### Improvements

- source_reuse 增加对 \_source 中数字类型的值进行复用压缩，可进一步降低 \_source 磁盘占用
- 改进 source_reuse 筛选字段的逻辑

## 1.5.0 (2023-09-08)

### Breaking changes

### Features

- 增加 sql 插件，支持使用 REST 接口和 JDBC 进行 SQL 查询
- 支持 sql 常用函数、包括数学函数、三角函数、日期函数、字符串函数、聚合函数等
- sql 语句可以嵌入全文检索
- 增加 jdbc 驱动，可以通过用户密码或证书连接到集群

### Bug fix

- 修复 knn 插件的配置项导致非 knn 索引的 setting 不能正常解析的 bug

### Improvements

## 1.4.0 (2023-07-21)

### Breaking changes

### Features

- 索引生命周期管理增加 wait_for_snapshot 操作，在删除索引之前，等待执行指定的快照管理策略，这样可以确保已删除索引的快照可用
- 增加 analysis-hanlp 分词插件
- 增加 jieba 分词插件

### Bug fix

- 修复启用 index.source_reuse 时，对复杂多层 json 的 source 字段 解析不正确的 bug

### Improvements

- 更新索引生命周期管理 api 文档，增加策略应用和更新说明，增加 wait_for_snapshot 说明
- 执行 initialize.sh 命令时增加初始化确认提示，是否将 admin 密码记录日志。

## 1.3.0 (2023-06-30)

### Breaking changes

### Features

- 增加 kNN 检索插件：
  - 新增 knn_nearest_neighbors query api
  - Mapping 新增 knn_dense_float_vector 和 knn_sparse_bool_vector 数据类型
  - 支持近似 kNN 搜索和精确 kNN 搜索

### Bug fix

### Improvements

- admin 用户默认由 initialize.sh 脚本生成随机密码,增强了安全性
- 增加适配 Windows 平台
- 增加 Docker 镜像

## 1.2.0 (2023-06-08)

### Breaking changes

### Features

- 正式发布快照生命周期管理 (SLM) API, 支持定时备份和删除快照，以及保留快照的个数
- 增加 跨集群复制 (Cross-cluster replication) 功能：
  - 支持手动或自动复制索引
  - 支持暂停和恢复复制索引
  - 支持取消指定索引的跨集群复制

### Bug fix

- security 模块修复缺少某些角色验证属性的问题

### Improvements

- 兼容 ES6.0 版本的索引

## 1.1.1 (2023-05-25)

### Breaking changes

### Features

### Bug fix

- 修复模板别名在某些场景不生效的 bug
- 防止 BigArray 在某些场景发生内存泄漏
- 修复 SourceValueFetcher 可能遗漏字段的 bug

### Improvements

- easysearch.yml 增加 elasticsearch.api_compatibility 配置项,
  兼容 logstash-oss, filebeat-oss, apm-server-oss 等 Elasticsearch 的客户端

## 1.1.0 (2023-05-12)

### Breaking changes

- Lucene 版本升级到 8.11.2

### Features

- 增加 ZSTD codec，引入 ZSTD 压缩算法，对存储字段，doc_values，词典进行压缩。
- 增加 index.source_reuse 索引级别配置，对 \_source 进一步压缩。
- 提供索引生命周期管理 ILM 模块的功能，绝大部分 API 兼容 Elasticsearch

### Bug fix

### Improvements

- 减少冗余日志输出。
- 减少 modules 模块整体大小

## 1.0.0 (2023-04-06)

### Features

- 兼容 Elasticsearch7.x
- 支持加密传输，权限控制等 security 相关功能
- 相比 Elasticsearch 更加轻量级

### Bug fix

### Improvements


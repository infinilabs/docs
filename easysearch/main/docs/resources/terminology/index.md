---
title: "术语表"
date: 0001-01-01
description: "Easysearch 文档中英文术语对照，统一用词规范。"
tags: ["术语", "Glossary"]
summary: "术语表 #  本页汇总文档中常用的中英文术语对照，方便写作时统一用词，也方便读者建立清晰的心智模型。
 一、数据结构与存储 #     英文术语 中文叫法 说明     Index 索引 逻辑上的数据集合，通常一类业务一组索引，可按时间/租户再拆前缀。   Document 文档 索引中的基本数据单元，以 JSON 形式表示。   Shard 分片 水平切分单位，number_of_shards 只在建索引时生效。   Primary Shard 主分片 负责接受写入并复制到副本。   Replica Shard 副本分片 / 副本 提供高可用与读扩展，副本数可在线调整。   Segment 段 Lucene 的不可变索引文件块，刷新/合并都围绕它展开。   Inverted Index 倒排索引 从词项到文档的映射结构，全文搜索的核心数据结构。   _source _source 文档原文 建议默认保留，是检索展示与重建索引的&quot;真相来源&quot;。   Stored Fields 存储字段 只在少数场景单独使用，更多依赖 _source + doc_values。   doc_values doc_values 列式存储 聚合与排序的核心支撑，应在大多数可聚合/排序字段上启用。   Fielddata fielddata 仅在 text 字段聚合/排序时使用，能不用尽量不用。   Translog 事务日志 写入操作的预写日志（WAL），保证 flush 前的数据不丢失。   Routing 路由 控制文档写入和查询时定向到特定分片的机制，默认按 _id 哈希。    二、Mapping 与文本分析 #     英文术语 中文叫法 说明     Mapping 映射 / Mapping 描述字段类型与索引规则，是一切查询/聚合行为的基础。   text Field 文本字段（text） 做全文检索，用分析器拆分为词项，不适合精确过滤/聚合。   keyword Field 关键字字段（keyword） 精确匹配、过滤、聚合、排序使用，不做分词。   integer / long / float / double 数值字段 数值类型字段，支持范围查询和聚合运算。   date Field 日期字段 支持多种日期格式，底层以毫秒时间戳存储。   boolean Field 布尔字段 仅存储 true/false 值。   geo_point 地理点 存储经纬度坐标，支持地理距离和区域查询。   geo_shape 地理形状 存储多边形、线段等复杂地理形状，支持空间关系查询。   nested 嵌套类型 保持对象数组中字段关联关系的特殊映射类型。   join (Parent/Child) 父子关系 同一索引内建立文档间的层级关系。   object 对象类型 JSON 对象映射为扁平化的点分字段名，不保持数组内对象边界。   knn_dense_float_vector 向量字段类型 用于存储 dense 浮点向量，支持近似最近邻搜索。   Multi-fields 多字段 / multi-fields 一份源数据多个视图，如 title + title."
---


# 术语表

本页汇总文档中常用的中英文术语对照，方便写作时统一用词，也方便读者建立清晰的心智模型。

---

## 一、数据结构与存储

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Index | 索引 | 逻辑上的数据集合，通常一类业务一组索引，可按时间/租户再拆前缀。 |
| Document | 文档 | 索引中的基本数据单元，以 JSON 形式表示。 |
| Shard | 分片 | 水平切分单位，`number_of_shards` 只在建索引时生效。 |
| Primary Shard | 主分片 | 负责接受写入并复制到副本。 |
| Replica Shard | 副本分片 / 副本 | 提供高可用与读扩展，副本数可在线调整。 |
| Segment | 段 | Lucene 的不可变索引文件块，刷新/合并都围绕它展开。 |
| Inverted Index | 倒排索引 | 从词项到文档的映射结构，全文搜索的核心数据结构。 |
| `_source` | `_source` 文档原文 | 建议默认保留，是检索展示与重建索引的"真相来源"。 |
| Stored Fields | 存储字段 | 只在少数场景单独使用，更多依赖 `_source` + `doc_values`。 |
| `doc_values` | doc_values 列式存储 | 聚合与排序的核心支撑，应在大多数可聚合/排序字段上启用。 |
| Fielddata | fielddata | 仅在 text 字段聚合/排序时使用，能不用尽量不用。 |
| Translog | 事务日志 | 写入操作的预写日志（WAL），保证 flush 前的数据不丢失。 |
| Routing | 路由 | 控制文档写入和查询时定向到特定分片的机制，默认按 `_id` 哈希。 |

## 二、Mapping 与文本分析

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Mapping | 映射 / Mapping | 描述字段类型与索引规则，是一切查询/聚合行为的基础。 |
| `text` Field | 文本字段（text） | 做全文检索，用分析器拆分为词项，不适合精确过滤/聚合。 |
| `keyword` Field | 关键字字段（keyword） | 精确匹配、过滤、聚合、排序使用，不做分词。 |
| `integer` / `long` / `float` / `double` | 数值字段 | 数值类型字段，支持范围查询和聚合运算。 |
| `date` Field | 日期字段 | 支持多种日期格式，底层以毫秒时间戳存储。 |
| `boolean` Field | 布尔字段 | 仅存储 true/false 值。 |
| `geo_point` | 地理点 | 存储经纬度坐标，支持地理距离和区域查询。 |
| `geo_shape` | 地理形状 | 存储多边形、线段等复杂地理形状，支持空间关系查询。 |
| `nested` | 嵌套类型 | 保持对象数组中字段关联关系的特殊映射类型。 |
| `join` (Parent/Child) | 父子关系 | 同一索引内建立文档间的层级关系。 |
| `object` | 对象类型 | JSON 对象映射为扁平化的点分字段名，不保持数组内对象边界。 |
| `knn_dense_float_vector` | 向量字段类型 | 用于存储 dense 浮点向量，支持近似最近邻搜索。 |
| Multi-fields | 多字段 / multi-fields | 一份源数据多个视图，如 `title` + `title.keyword`。 |
| Dynamic Mapping | 动态映射 | 自动推断新字段类型，推荐配合 `dynamic_templates` 使用。 |
| Dynamic Template | 动态模板 | 按字段名模式或数据类型自动套用指定的字段映射规则。 |
| Analyzer | 分析器 | 由字符过滤器、分词器和词元过滤器组成，可分索引时/查询时。 |
| Tokenizer | 分词器 | 决定如何切词，如 `standard`、语言专用分词器等。 |
| Token Filter | 词元过滤器 | 大小写、词干、停用词、同义词、n-gram 等都在这里实现。 |
| Char Filter | 字符过滤器 | 在分词前对原始文本做预处理（如 HTML 剥离、字符替换）。 |
| Normalizer | 归一化器 | 仅用于 keyword 字段的字符级标准化（小写、Unicode 折叠等）。 |
| Stemming | 词干提取 | 通过归并词形提升召回，需要结合 `keyword_marker` 等控制风险。 |
| Stopwords | 停用词 | 主要价值是性能，现代实践中应谨慎、少量地使用。 |
| Synonyms | 同义词 | 扩大召回，强调规则版本化与可回滚。 |

## 三、查询与相关性

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Query DSL | 查询 DSL | Easysearch 基于 JSON 的查询语言，支持全文、精确、复合等各类查询。 |
| Query Context | 查询上下文 | 参与评分。 |
| Filter Context | 过滤上下文 | 不参与评分，易缓存。 |
| BM25 | BM25 算法 | 默认相关性评分算法，替代旧版 TF/IDF。参数 k1=1.2、b=0.75。 |
| `bool` Query | bool 查询 | 通过 must / should / filter / must_not 组合语义。 |
| `match` Query | match 查询 | 针对 text 字段，内部会做分析，不等同于 term。 |
| `term` / `terms` Query | term / terms 查询 | 精确值查询，用于 keyword/数值/日期字段。 |
| `range` Query | 范围查询 | 数值/日期字段常用，在 text 字段上属于昂贵查询。 |
| `match_phrase` / `slop` | 短语查询 / slop | 用 positions 做邻近匹配，成本高于普通 match。 |
| `multi_match` | multi_match 多字段查询 | `best_fields`、`most_fields`、`cross_fields` 几种模式。 |
| `dis_max` + `tie_breaker` | DisMax 查询 | 用于"字段竞争"型多字段搜索（如 title vs body）。 |
| `function_score` | function_score 加权 | 用于叠加业务信号（热度、时间衰减等）。 |
| `nested` Query | 嵌套查询 | 查询嵌套对象字段，保持对象边界完整性。 |
| `has_child` / `has_parent` | 父子查询 | 基于父子关系跨文档查询。 |
| Span Query | Span 查询 | 词项级精细控制查询族，可精确控制词项间距离和位置。 |
| Fuzziness / Fuzzy Matching | 模糊匹配 | 拼写容错，适合作兜底召回。 |
| Rescore | 查询重打分 | 对初始 Top-N 结果用更复杂的查询重新打分的二阶段策略。 |
| Field Collapsing | 结果折叠 | 按字段值对搜索结果分组去重，每组只返回 Top-N。 |
| `search_after` | search_after 深分页 | 用于替代大 `from`，建议在长列表场景中使用。 |
| Scroll / PIT | 滚动 / 时间点搜索 | 用于批量扫描/导出，不适合用户界面分页。 |
| Async Search | 异步搜索 | 将大查询提交到后台执行，客户端可轮询获取进度和结果。 |
| Highlight | 高亮 | 结果展示层的增强能力。 |
| Suggesters / Autocomplete | 建议与纠错 / 自动补全 | 拼写纠错和即时补全。 |
| Search Template | 搜索模板 | 使用 Mustache 模板参数化搜索查询，简化客户端代码。 |
| `_explain` | 评分解释 | 返回单条文档的评分计算过程，用于相关性调试。 |
| `_preference` | 查询偏好 | 控制搜索请求路由到特定分片或节点的参数。 |
| `track_total_hits` | 总命中数追踪 | 控制是否精确统计查询的总命中文档数，默认上限 10000。 |
| Profile API | 查询分析 API | 分析查询各阶段耗时，用于定位性能瓶颈。 |
| Term Vectors | 词项向量 | 获取文档中特定字段的词频、位置、偏移量等信息。 |

## 四、聚合与分析

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Aggregation (aggs) | 聚合 | 对文档集合进行分组统计和指标计算的功能。 |
| Bucket Aggregation | 桶聚合 | 将文档按规则分组到不同的"桶"中（如 terms、date_histogram、range）。 |
| Metric Aggregation | 指标聚合 | 在桶内对数值字段做统计计算（如 avg、sum、min、max、cardinality）。 |
| Pipeline Aggregation | 管道聚合 | 对其他聚合的输出结果做二次计算（如 derivative、moving_avg、cumulative_sum）。 |
| `terms` Aggregation | terms 聚合 | 按字段值分桶，返回每个唯一值的文档计数。 |
| `date_histogram` | 日期直方图 | 按时间间隔（如小时/天/月）分桶，时序分析的核心聚合。 |
| `histogram` | 直方图聚合 | 按固定数值区间分桶。 |
| `range` Aggregation | 范围聚合 | 按自定义数值/日期范围分桶。 |
| `composite` Aggregation | 复合聚合 | 支持分页的多维聚合，适合高基数场景。 |
| `filter` / `filters` Aggregation | 过滤器聚合 | 用查询条件定义桶，按条件分组统计。 |
| `global` Aggregation | 全局聚合 | 忽略查询条件，在全量文档上做统计。 |
| `nested` / `reverse_nested` Agg | 嵌套 / 反嵌套聚合 | 对嵌套对象字段进行聚合或从嵌套返回父文档上下文。 |
| `sampler` / `diversified_sampler` | 采样聚合 | 对高频桶采样以提升聚合性能或多样性。 |
| `cardinality` | 基数 / 去重计数 | 估算字段中唯一值的数量（基于 HyperLogLog++）。 |
| `percentiles` | 百分位数 | 计算数值字段的百分位分布（P50、P95、P99 等）。 |
| `top_hits` | Top Hits 聚合 | 在每个桶内返回最相关的文档（常配合 terms 聚合使用）。 |
| `significant_terms` | 显著词项 | 发现统计上异常突出的词项，用于趋势发现和异常检测。 |

## 五、摄取与搜索管道

### 摄取管道（Ingest Pipeline）

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Ingest Pipeline | 摄取管道 | 文档写入前的预处理链路，由有序处理器列表组成。 |
| Processor | 处理器 | 摄取管道中的单个处理单元（如 set、remove、grok、script 等）。 |
| `grok` Processor | Grok 处理器 | 使用正则模式从非结构化文本中提取结构化字段。 |
| `dissect` Processor | Dissect 处理器 | 基于分隔符从文本中提取字段，比 grok 更轻量。 |
| `script` Processor | 脚本处理器 | 使用 Painless 脚本对文档做自定义转换。 |
| `geoip` Processor | GeoIP 处理器 | 根据 IP 地址解析地理位置信息。 |
| `text_embedding` Processor | 文本向量化处理器 | 在写入时自动调用 Embedding 模型将文本转为向量。 |
| `default_pipeline` | 默认管道 | 索引设置中指定的默认摄取管道。 |
| `final_pipeline` | 最终管道 | 索引设置的强制管道，无法被客户端绕过。 |
| `on_failure` | 失败处理 | 处理器失败时执行的备用处理器列表。 |
| `_simulate` | 模拟管道 | 模拟管道执行以测试处理逻辑，不实际写入。 |

### 搜索管道（Search Pipeline）

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Search Pipeline | 搜索管道 | 拦截搜索请求和响应的处理链路，支持查询重写和结果增强。 |
| Request Processor | 请求处理器 | 在查询执行前对原始查询进行修改和增强。 |
| Response Processor | 响应处理器 | 对搜索返回结果进行后处理和增强。 |
| Search Phase Results Processor | 搜索阶段结果处理器 | 在搜索阶段之间对合并结果进行处理（如 RRF 重排序）。 |

## 六、索引管理与生命周期

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Index Alias | 索引别名 | 指向一个或多个索引的虚拟名称，用于无感迁移和蓝绿切换。 |
| Write Alias / `is_write_index` | 写别名 | 别名标记为写入目标，配合 rollover 实现无感索引滚动。 |
| Index Template | 索引模板 | 新索引创建时自动套用的 settings/mappings/aliases 预配置。 |
| Component Template | 组件模板 | 可复用的模板构建块，被可组合索引模板引用。 |
| Data Stream | 数据流 | 面向时序数据的抽象，内部由多个 backing index 组成，简化 rollover 管理。 |
| Backing Index | 后备索引 | 数据流内部的底层索引，由数据流自动创建和管理。 |
| ILM (Index Lifecycle Management) | 索引生命周期管理 | 自动化管理索引从创建到删除的全生命周期策略。 |
| ILM Phase (hot/warm/cold/delete) | 生命周期阶段 | ILM 策略中的生命周期阶段，按数据访问频率逐级迁移。 |
| Hot-Warm-Cold Architecture | 热温冷架构 | 按数据访问频率将索引分配到不同性能层级的节点。 |
| Rollover | 索引滚动 | 当索引达到年龄/大小/文档数阈值时，自动滚动到新索引。 |
| Rollup | 数据汇总 / 上卷 | 将细粒度历史数据聚合为粗粒度格式以降低存储成本。 |
| Transform | 数据转换 | 将源索引的数据通过聚合转换为新索引的汇总数据。 |
| Clone / Shrink / Split | 克隆 / 缩小 / 拆分 | 调整索引分片结构的三个 API。 |
| Open / Close Index | 打开 / 关闭索引 | 关闭索引不消耗资源但保留数据，可随时重新打开。 |
| Index Block | 索引限制 | 限制索引的读/写行为（如 `index.blocks.write`）。 |
| Snapshot / Restore | 快照 / 恢复 | 官方推荐的数据安全与恢复手段。 |
| SLM (Snapshot Lifecycle Management) | 快照生命周期管理 | 自动化的快照创建与清理策略。 |
| Index Codec / ZSTD | 索引编码 / ZSTD 压缩 | 控制索引存储字段的压缩算法，ZSTD 在压缩比和速度间取得平衡。 |
| `source_reuse` | Source 复用 | 去除 `_source` 中与 doc_values 重复的部分以减小索引大小。 |

## 七、常用 API

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Bulk API | 批量 API | 在单个请求中执行多个 index/create/update/delete 操作。 |
| NDJSON | 逐行 JSON | Bulk API 使用的格式，每行一个 JSON 对象。 |
| `_reindex` | 重建索引 | 将文档从源索引复制到目标索引，支持查询过滤和脚本转换。 |
| `_update_by_query` | 按查询更新 | 按查询条件批量更新匹配文档，支持脚本修改。 |
| `_delete_by_query` | 按查询删除 | 按查询条件批量删除匹配文档。 |
| CAT API | CAT 接口 | 以易读表格格式返回集群状态信息的一组 API。 |
| Task API | 任务 API | 查看和管理集群中正在运行的任务。 |
| `_analyze` | 分析测试 | 测试分析器对给定文本的分词结果。 |

## 八、分布式与集群管理

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Cluster | 集群 | 一组协同工作的节点，共享相同的 `cluster.name`。 |
| Node | 节点 | 集群中的一个 Easysearch 实例。 |
| `node.roles` | 节点角色 | 定义节点承担的职责（master/data/ingest/search 等）。 |
| Master Node | 主节点 | 管理集群状态、索引创建/删除、分片分配的专用节点。 |
| Data Node | 数据节点 | 存储和搜索数据，执行本地分片操作的工作节点。 |
| Ingest Node | 摄取节点 | 执行 Ingest Pipeline 预处理的节点。 |
| Coordinating Node | 协调节点 | 接收请求、拆分到各分片并汇总结果。每个节点都可充当协调节点。 |
| Search Node | 搜索节点 | 角色为 `search` 的专用节点。 |
| `remote_cluster_client` | 远程集群客户端 | 节点角色，允许连接和访问远程集群。 |
| Cluster State | 集群状态 | 集群元数据（mapping、settings、管道定义等），由主节点管理并分发。 |
| Cluster Health | 集群健康 | 集群的整体状态：green（全部正常）/ yellow（副本缺失）/ red（主分片缺失）。 |
| Node Discovery | 节点发现 | 节点加入集群的自动发现机制。 |
| `seed_hosts` | 种子节点 | 节点发现时的初始联系节点列表。 |
| Shard Allocation | 分片分配 | 主节点将分片分配到数据节点的决策过程。 |
| Rebalancing | 分片再平衡 | 集群自动在节点间移动分片以均衡负载。 |
| Split Brain | 脑裂 | 网络分区导致多个主节点并存的异常状态。 |
| Refresh | 刷新（refresh） | 控制近实时可见性，新写入变为可搜索，不等价于持久化。 |
| Flush | 刷新到磁盘（flush） | 将段持久化到磁盘并清空事务日志，主要保证数据安全。 |
| Force Merge | 强制合并 | 手动触发段合并，将多个小段合并为少量大段。仅用于只读索引。 |
| Merge | 段合并（merge） | 合并段，带来磁盘/IO 峰值，是调优重点。 |

## 九、性能调优

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Circuit Breaker | 断路器 | 内存保护机制，在数据加载前估算内存需求，超限则中止操作。 |
| Fielddata Circuit Breaker | Fielddata 断路器 | 限制 fielddata 内存使用（默认不超过堆的 40%）。 |
| Request Circuit Breaker | 请求断路器 | 限制单个请求使用的内存（默认不超过堆的 60%）。 |
| Query Cache | 查询缓存 | 缓存 filter 子句结果的位集，加速重复查询。 |
| Request Cache | 请求缓存 | 缓存不变索引上的完整聚合/计数结果。 |
| `refresh_interval` | 刷新间隔 | 控制新写入文档变为可搜索的延迟（默认 1s）。 |
| Slow Log | 慢日志 | 记录超过阈值的搜索/索引操作，用于性能排查。 |
| Write Throttling | 写入限流 | 在节点/索引/分片三个层级控制写入速率。 |
| Disk Watermark | 磁盘水位线 | 磁盘使用率阈值，超过后限制分片分配或变为只读。 |
| Thread Pool | 线程池 | 各类操作（搜索、写入、管理等）的并发线程管理。 |
| Indexing Buffer | 索引缓冲区 | 新文档写入时的内存缓冲区，满后触发 refresh。 |
| Bulk Queue | 批量队列 | 写入请求的排队区域，队满时返回 rejected execution。 |

## 十、安全与多租户

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Authentication | 认证 | 推荐接企业 SSO/LDAP/AD/OIDC。 |
| Authorization / Access Control | 授权 / 权限控制 | 分集群级、索引级、文档/字段级。 |
| Role / Role Mapping | 角色 / 角色映射 | 安全模块内部权限载体，与外部后端角色映射。 |
| Backend Role | 后端角色 | 来自外部认证系统（如 LDAP 组）的角色字符串，用于角色映射。 |
| Action Group | 权限组 | 将多个权限打包复用（如 `read`、`write`、`crud`）。 |
| Document-level Security (DLS) | 文档级安全 | 按标签/租户过滤可见文档。 |
| Field-level Security (FLS) | 字段级安全 | 隐藏敏感字段（PII、密钥等）。 |
| Field Masking | 字段脱敏 | 对敏感字段做哈希或正则替换，用户只看到脱敏值。 |
| Multi-tenancy | 多租户 | 按索引隔离 vs 按字段标记 + 文档级安全两种主模式。 |
| TLS / SSL | 传输加密 | Transport 层（节点间）和 HTTP 层（客户端）通信加密。 |
| Admin Certificate | 管理员证书 | 具有最高权限的客户端证书，用于管理安全配置。 |
| Audit Log | 审计日志 | 记录认证、授权、高危操作等安全事件，支持合规审计。 |
| Run As (Impersonation) | 身份模拟 | 允许有权限的用户以另一用户身份执行操作。 |
| National Encryption (SM2/SM3/SM4) | 国密算法 | 国产密码算法全量支持，满足信创与等保合规。 |

## 十一、AI 与向量搜索

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Embedding | 向量表示 / Embedding | 由模型生成，存储在向量字段中。 |
| kNN / ANN Search | 向量 / 近似最近邻搜索 | 通过近似算法在高维向量空间中快速找到最相似的结果。 |
| Hybrid Search (BM25+Vector) | 混合检索 | BM25 全文搜索 + 向量搜索的组合策略。 |
| RRF (Reciprocal Rank Fusion) | 倒数排名融合 | 融合多路检索结果排序的算法，用于混合搜索。 |
| RAG | 检索增强生成（RAG） | Easysearch 作为检索层，LLM 作为生成层。 |
| Multimodal Search | 多模态搜索 | 跨文本/图片/音频等数据形态的统一向量检索。 |

## 十二、SQL 访问

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| SQL Plugin | SQL 插件 | 允许使用标准 SQL 语法查询 Easysearch，翻译为原生 DSL 执行。 |
| `_sql` Endpoint | SQL 端点 | 执行 SQL 查询的 REST API 入口。 |
| `_sql/_explain` | SQL 转译 | 将 SQL 翻译为 Easysearch DSL 但不执行，用于调试和学习。 |
| Cursor (SQL) | 游标 | SQL 分页机制，支持大结果集的逐批获取。 |
| JDBC Driver | JDBC 驱动 | 纯 Java 驱动程序，将 JDBC 调用转换为 Easysearch SQL REST API。 |

## 十三、跨集群

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| CCR (Cross-Cluster Replication) | 跨集群复制 | 将数据从 Leader 集群实时同步到 Follower 集群，用于容灾和读写分离。 |
| Leader Index | 领导者索引 | CCR 中作为数据源的可写索引。 |
| Follower Index | 跟随者索引 | CCR 中的只读数据副本索引。 |
| Auto-Follow | 自动跟随 | 按模式自动对新建的匹配索引启动跨集群复制。 |
| CCS (Cross-Cluster Search) | 跨集群搜索 | 一次查询覆盖多个远程集群，无需数据迁移。 |
| Remote Cluster | 远程集群 | 通过种子节点连接的外部集群。 |

## 十四、脚本与扩展

| 英文术语 | 中文叫法 | 说明 |
|----------|----------|------|
| Painless | Painless 脚本语言 | Easysearch 内置的安全脚本语言，用于自定义评分、处理和转换。 |
| Stored Script | 存储脚本 | 保存在集群状态中的可复用脚本，通过 ID 引用。 |
| Plugin | 插件 | 扩展 Easysearch 功能的可安装组件。 |
| Module | 模块 | 内置集成的功能组件，无需单独安装。 |
| Rule Engine | 规则引擎 | 高性能实时规则匹配，在写入时自动完成关键词检测和打标。 |



---
title: "测试与验证"
date: 0001-01-01
description: "插件测试框架、Gradle 任务、运行态验证与常见问题排查。"
summary: "测试与验证 #  本文档聚焦 Easysearch 插件的测试与验证，包括测试基类的选择、Gradle 任务、运行态验证以及常见问题排查。插件初始化、打包安装和本地开发闭环请参考： 插件开发入门。
引入 test-framework 依赖 #  Easysearch 官方提供了 test-framework 包，内置随机化测试、嵌入式节点、Hamcrest 断言等工具，所有测试基类均来自这个包。
在插件项目中添加依赖 #  若你的插件项目与 Easysearch 源码在同一 Gradle 多项目构建中，使用项目引用：
dependencies { testImplementation project(&#39;:test:framework&#39;) } 若你的插件是独立项目（如从官方模板克隆），使用 Maven 坐标：
dependencies { testImplementation &#34;com.infinilabs.easysearch.test:framework:${easysearch_version}&#34; }  同时确保 repositories 中包含 Easysearch 制品库，或将 test-framework 的 jar 发布到本地 Maven（./gradlew publishToMavenLocal）。
 测试基类与选择指南 #  Easysearch 提供了多个测试基类，适用于不同的测试场景。
测试基类概览 #     基类 适用场景 常见任务 启动开销     EasySearchTestCase 纯单元测试，无需启动节点 test 最低   ESSingleNodeTestCase 单节点节点内测试，验证插件加载和组件交互 通常是 test 中等   ESIntegTestCase 多节点集群级测试，验证分布式行为 通常是 internalClusterTest / icTest 较高   EasySearchRestTestCase 通过 REST API 测试 HTTP 行为 integTest / yamlRestTest 较高    如何选择测试类型？ #  单元测试是首选：大多数情况下，单元测试更易于理解、更容易复现，且不易受到与被测功能无关的代码变更影响。"
---


# 测试与验证

本文档聚焦 Easysearch 插件的测试与验证，包括测试基类的选择、Gradle 任务、运行态验证以及常见问题排查。插件初始化、打包安装和本地开发闭环请参考：[插件开发入门]({{< relref "./getting-started.md" >}})。

## 引入 test-framework 依赖

Easysearch 官方提供了 `test-framework` 包，内置随机化测试、嵌入式节点、Hamcrest 断言等工具，所有测试基类均来自这个包。

### 在插件项目中添加依赖

若你的插件项目与 Easysearch 源码在同一 Gradle 多项目构建中，使用项目引用：

```groovy
dependencies {
    testImplementation project(':test:framework')
}
```

若你的插件是独立项目（如从官方模板克隆），使用 Maven 坐标：

```groovy
dependencies {
    testImplementation "com.infinilabs.easysearch.test:framework:${easysearch_version}"
}
```

> 同时确保 `repositories` 中包含 Easysearch 制品库，或将 `test-framework` 的 jar 发布到本地 Maven（`./gradlew publishToMavenLocal`）。

## 测试基类与选择指南

Easysearch 提供了多个测试基类，适用于不同的测试场景。

### 测试基类概览

| 基类 | 适用场景 | 常见任务 | 启动开销 |
|---|---|---|---|
| `EasySearchTestCase` | 纯单元测试，无需启动节点 | `test` | 最低 |
| `ESSingleNodeTestCase` | 单节点节点内测试，验证插件加载和组件交互 | 通常是 `test` | 中等 |
| `ESIntegTestCase` | 多节点集群级测试，验证分布式行为 | 通常是 `internalClusterTest` / `icTest` | 较高 |
| `EasySearchRestTestCase` | 通过 REST API 测试 HTTP 行为 | `integTest` / `yamlRestTest` | 较高 |

### 如何选择测试类型？

**单元测试是首选**：大多数情况下，单元测试更易于理解、更容易复现，且不易受到与被测功能无关的代码变更影响。

**何时使用 `ESSingleNodeTestCase`**：历史上 Easysearch 的组件难以独立设置，导致大量集成测试而单元测试很少。`ESSingleNodeTestCase` 提供了一个折中方案——它可以轻松启动单节点并获取难以实例化的组件（如 `IndicesService`）。**只要可行，应优先使用单元测试**。

**何时使用 `ESIntegTestCase`**：许多测试继承 `ESIntegTestCase`，主要是因为早期 Easysearch 测试大多采用这种方式。但这类测试的复杂性往往使调试变得困难。**只有当被测功能与集群行为紧密相关时**（如分片分配、副本同步），才建议使用 `ESIntegTestCase`。

**总结**：新功能应主要提供单元测试，必要时辅以 REST 测试或单节点集成测试来验证集成行为。

## Gradle 任务与测试框架映射

源码里的任务注册大致分成三类：

| 任务 | 来源 | 默认测试目录 | 适合什么测试 |
|---|---|---|---|
| `test` | Gradle Java 测试任务 | `src/test/java` | 单元测试，以及少量 `ESSingleNodeTestCase` |
| `internalClusterTest` | `easysearch.internal-cluster-test` 插件创建 | `src/internalClusterTest/java` | `ESIntegTestCase` 这类节点内/集群内测试 |
| `icTest` | `internalClusterTest` 的别名 | - | 只是更短的命令入口 |
| `integTest` | `easysearch.rest-test` 或项目自定义 `RestIntegTestTask` | 取决于项目配置 | Java REST 集成测试 |
| `yamlRestTest` | `easysearch.yaml-rest-test` 插件创建 | `src/yamlRestTest/java` 与 `src/yamlRestTest/resources` | YAML REST 测试 |
| `check` | Gradle 聚合校验任务 | - | 汇总当前项目已接入的校验/测试任务 |
| `run` | `RunTask` + `testClusters.runTask` | - | 启动一个前台 Easysearch 进程用于本地联调 |

几点需要特别注意：

- `easysearch.internal-cluster-test` 会新增 `internalClusterTest` source set，并额外注册 `icTest` 别名任务。
- `integTest` 和 `yamlRestTest` 都基于 `RestIntegTestTask`，默认会通过 `testClusters` 自动创建并启动测试集群，不是默认去连一个手工准备的外部集群。
- 只有当你显式提供 `tests.rest.cluster`、`tests.cluster`、`tests.clustername` 等系统属性时，REST 测试才会改为连接外部集群。
- 文档中的“集成测试”最好区分为两类：节点内测试（`ESSingleNodeTestCase`/`ESIntegTestCase`）和 REST 集成测试（`EasySearchRestTestCase`/YAML REST）。

### `check` 任务

`check` 是聚合入口，但它依赖哪些任务，取决于项目应用了哪些插件：

- `test` 是 Java 项目的基础校验任务。
- `internalClusterTest` 会在 `easysearch.internal-cluster-test` 里自动挂到 `check`。
- `integTest` 会在 `easysearch.rest-test` 里自动挂到 `check`。
- `yamlRestTest` 会在 `easysearch.yaml-rest-test` 里自动挂到 `check`。
- 某些项目还会额外挂接自定义任务，例如 `client/rest-high-level` 会把 `asyncIntegTest` 也挂到 `check`。

因此：

```bash
./gradlew check
```

通常适合在提交前做“当前项目的完整校验”，但它不是固定只跑某三四个任务，而是由构建脚本汇总当前项目接入的测试和校验任务。

### `run` 任务

`run` 适合做本地联调，而不是跑自动化测试。源码里有两种常见接法：

- 根工程的 [gradle/run.gradle](/Users/user/gitea/easysearch/gradle/run.gradle) 注册了 `run`，用 `testClusters.runTask` 启动一个前台 Easysearch。
- 插件工程的 [PluginBuildPlugin.groovy](/Users/user/gitea/easysearch/buildSrc/src/main/groovy/org/easysearch/gradle/plugin/PluginBuildPlugin.groovy) 也会注册 `run`，并先依赖 `bundlePlugin`，再把当前插件或模块自动装入 `runTask` 集群。

这意味着对插件开发来说，`run` 很适合做“构建插件包并启动带该插件的本地节点”：

```bash
./gradlew run
```

`RunTask` 还支持一些常用参数：

```bash
# 保留数据目录
./gradlew run --preserve-data

# 指定数据目录
./gradlew run --data-dir /tmp/easysearch-dev

# 以 JVM 调试模式启动
./gradlew run --debug-server-jvm
```

此外，`run` 会读取以 `tests.es.` 为前缀的系统属性，并把它们转成节点设置。例如：

```bash
./gradlew run -Dtests.es.node.attr.rack=r1
```

### 重构代码以提高可测试性

不幸的是，很多代码仍然难以进行单元测试——有时是因为类依赖过多难以实例化，有时是因为 API 契约使测试难以编写。**使代码更易于单元测试的重构是值得鼓励的**。

良好的重构示例：
- 减少对外部难以实例化对象（如 `IndexShard` 或 `SearchContext`）的依赖
- 将基于时间的驱逐逻辑从缓存内部移到外部，便于断言缓存内容
- 通过构造函数注入依赖，而非使用全局状态或静态方法

## 单元测试（EasySearchTestCase）

继承 `EasySearchTestCase` 可获得随机化测试、内置断言工具等能力，**无需启动 Easysearch 节点**：

```java
import org.easysearch.test.EasySearchTestCase;

public class MyTokenFilterTests extends EasySearchTestCase {

    public void testThrowsOnNullInput() {
        String term = randomAlphaOfLength(8);
        IllegalArgumentException ex = expectThrows(
            IllegalArgumentException.class,
            () -> new MyTokenizer(null, term)
        );
        assertTrue(ex.getMessage().contains(term));
    }
}
```

**常用工具方法**（均来自 `EasySearchTestCase` / `LuceneTestCase`）：

| 方法 | 说明 |
|---|---|
| `randomAlphaOfLength(n)` | 生成长度为 n 的随机字母字符串 |
| `randomAlphaOfLengthBetween(min, max)` | 生成随机长度字母字符串 |
| `randomIntBetween(min, max)` | 生成随机整数 |
| `randomBoolean()` | 生成随机布尔值 |
| `randomFrom(collection)` | 从集合中随机取一个元素 |
| `expectThrows(ExType.class, () -> ...)` | 断言代码块抛出指定异常 |

运行：

```bash
./gradlew test
./gradlew test --tests "com.example.myplugin.MyTokenFilterTests"
```

## 集成测试（ESSingleNodeTestCase）

通过 `ESSingleNodeTestCase` 启动嵌入式单节点，验证插件在真实节点中的行为。测试类之间共享同一个节点实例，每个测试方法结束后自动清理索引。

```java
import org.easysearch.common.settings.Settings;
import org.easysearch.plugins.Plugin;
import org.easysearch.test.ESSingleNodeTestCase;

import java.util.Collection;
import java.util.List;

public class MyPluginIntegrationTests extends ESSingleNodeTestCase {

    /**
     * 声明需要加载的插件，框架启动节点时会自动注册。
     */
    @Override
    protected Collection<Class<? extends Plugin>> getPlugins() {
        return List.of(MyPlugin.class);
    }

    /**
     * 覆盖节点设置（可选）。
     */
    @Override
    protected Settings nodeSettings() {
        return Settings.builder()
            .put(super.nodeSettings())
            .put("my.plugin.enabled", true)
            .build();
    }

    public void testPluginWorksInsideSingleNode() {
        createIndex("test", Settings.builder()
            .put("index.number_of_shards", 1)
            .build());
        assertNotNull(client());
        assertNotNull(getInstanceFromNode(MyPluginService.class));
    }
}
```

**ESSingleNodeTestCase 常用工具**：

| 方法 | 说明 |
|---|---|
| `client()` | 获取节点内部 Client |
| `createIndex(name)` | 创建索引（自动等待变绿） |
| `createIndex(name, settings)` | 创建带配置的索引 |
| `getInstanceFromNode(Class<T>)` | 从 Guice 容器获取内部服务实例 |
| `node()` | 获取底层 Node 对象 |

运行：

```bash
./gradlew test
./gradlew test --tests "com.example.myplugin.MyPluginIntegrationTests"
```

> `ESSingleNodeTestCase` 只是测试基类，不会自动把测试放进 `integTest`。如果类位于默认的 `src/test/java`，通常还是由 `test` 任务执行。

## 多节点集成测试（ESIntegTestCase / internalClusterTest）

`ESIntegTestCase` 会在 JVM 内启动并管理测试集群，适合验证分片分配、副本同步、滚动重启后的行为等集群级逻辑。仓库中的这类测试通常放在 `src/internalClusterTest/java`，并由 `internalClusterTest` 任务执行；应用了 `easysearch.internal-cluster-test` 后，还可以用更短的别名 `icTest`。

```java
import org.easysearch.action.admin.cluster.node.info.NodesInfoResponse;
import org.easysearch.action.admin.cluster.node.info.PluginsAndModules;
import org.easysearch.plugins.Plugin;
import org.easysearch.test.ESIntegTestCase;
import org.easysearch.test.ESIntegTestCase.ClusterScope;
import org.easysearch.test.ESIntegTestCase.Scope;

import java.util.Collection;
import java.util.List;

@ClusterScope(scope = Scope.TEST, numDataNodes = 2)
public class MyPluginClusterIT extends ESIntegTestCase {

    @Override
    protected Collection<Class<? extends Plugin>> nodePlugins() {
        return List.of(MyPlugin.class);
    }

    public void testPluginLoadedOnAllNodes() {
        NodesInfoResponse nodesInfo =
            client().admin().cluster().prepareNodesInfo().addMetric("plugins").get();

        nodesInfo.getNodes().forEach(nodeInfo -> {
            PluginsAndModules pm = nodeInfo.getInfo(PluginsAndModules.class);
            assertNotNull(pm);
            assertTrue(
                "节点 " + nodeInfo.getNode().getName() + " 未加载插件",
                pm.getPluginInfos().stream().anyMatch(p -> p.getName().equals("my-plugin"))
            );
        });
    }
}
```

运行：

```bash
./gradlew internalClusterTest
./gradlew icTest --tests "com.example.myplugin.MyPluginClusterIT"
```

## Java REST 集成测试（integTest）

如果你的测试继承 `EasySearchRestTestCase`，通常由 `integTest` 执行。根据 `RestTestBasePlugin` 的配置，`integTest` 默认会通过 `testClusters` 创建名为 `integTest` 的测试集群，并把其地址注入 `tests.rest.cluster`、`tests.cluster`、`tests.clustername` 等系统属性。

这意味着：

- 默认情况下，`./gradlew integTest` 会先拉起由构建脚本管理的测试集群，再执行 REST 测试。
- 只有在你显式传入外部集群参数时，测试才会改为连接外部集群。
- 如果项目是插件/模块工程，`RestTestUtil` 还会把 `bundlePlugin` 产物自动装进测试集群。

示例：

```bash
./gradlew integTest
./gradlew integTest --tests "com.example.myplugin.MyPluginRestIT"
```

## YAML REST 测试

在 `src/yamlRestTest/resources/rest-api-spec/test/` 下写 YAML 测试，验证 HTTP 接口行为；若使用自定义 API 名称（如 `my_plugin.status`），需先在 `rest-api-spec/api` 定义对应 API：

```yaml
# my_plugin/10_basic.yml
---
"Test my plugin status endpoint":
  - do:
      my_plugin.status: {}
  - match: { status: "ok" }
  - match: { plugin: "my-plugin" }
```

运行：

```bash
./gradlew yamlRestTest
```

`yamlRestTest` 与 `integTest` 一样，默认也会通过 `testClusters` 启动测试集群，而不是直接依赖一个外部已存在的集群。

## 测试最佳实践与常见误区

### 何时使用随机化测试

**正确使用随机化**：随机化应仅用于**不预期影响功能行为**的参数，例如：
- 索引的分片数量（不应影响聚合结果）
- 存储类型（`niofs` vs `mmapfs`，不应影响查询结果）
- 字段名称的随机生成（测试通用性）

这类随机化有助于提高信心，确保代码不依赖于某个组件的实现细节。

**错误使用随机化**：不要用随机化替代明确的测试覆盖。例如，如果功能在单分片和多分片（2+）下走不同代码路径，**不应**只测试随机分片数，而应分别编写：
- 一个测试明确使用 1 个分片
- 另一个测试明确使用 2+ 个分片

### 多线程测试注意事项

多线程测试通常由于线程间操作顺序的不确定性而难以复现。**在多线程测试中随意添加随机化通常会使问题更糟**。

建议：
- 减少多线程测试中的随机化使用
- 使用明确的同步机制（如 `CountDownLatch`）控制执行顺序
- 避免依赖线程特定的时序假设

### 测试数据管理

- **保持测试独立**：每个测试应独立设置所需数据，不依赖其他测试的执行顺序
- **及时清理**：利用测试框架的自动清理机制（如 `ESSingleNodeTestCase` 在每个测试后清理索引）
- **避免硬编码**：使用随机值生成测试数据，避免与特定实现细节耦合

## 常见问题排查

### 插件安装失败

```bash
# 确认 zip 内包含描述文件
unzip -l build/distributions/my-plugin-0.1.0.zip | grep plugin-descriptor
```

- 没有 `plugin-descriptor.properties` → 检查 `build.gradle` 的 `esplugin` 块
- 版本不匹配 → 确认 `easysearch.version` 与运行环境一致

### 插件加载后无响应

```bash
# 查看启动日志
tail -f logs/easysearch.log | grep -i "myplugin\|error\|exception"
```

- `classname` 与实际类名不一致 → 检查 `plugin-descriptor.properties`
- 依赖缺失 → 使用 `shadow` 插件将依赖打包进 jar

### 分析器不生效

```bash
# 直接测试分析器输出
curl -s -X POST http://localhost:9200/my-index/_analyze \
  -H 'Content-Type: application/json' \
  -d '{"analyzer":"my_analyzer","text":"测试文本"}'
```

### 测试运行失败

```bash
# 运行单个测试类以隔离问题
./gradlew test --tests "com.example.myplugin.MySpecificTest"

# 运行单个 internalClusterTest 用例
./gradlew icTest --tests "com.example.myplugin.MyPluginClusterIT"

# 运行单个 REST 用例
./gradlew integTest --tests "com.example.myplugin.MyPluginRestIT"

# 查看详细输出
./gradlew test --info

# 固定 seed / 调整堆大小，便于复现问题
./gradlew integTest -Dtests.seed=123456 -Dtests.heap.size=512m
```

## 验收标准

在提交或发布前确保：

1. `./gradlew check` 通过（覆盖当前项目已挂到 `check` 的单元测试、`internalClusterTest`、`integTest`、`yamlRestTest` 以及其他校验任务）
2. `/_cat/plugins?v` 能看到插件信息
3. 核心功能有至少一条可复现的 `curl` 请求

## 下一步

- [插件发布与分享]({{< relref "./publishing.md" >}})


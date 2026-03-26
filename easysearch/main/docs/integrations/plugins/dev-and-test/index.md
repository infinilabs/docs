---
title: "开发与测试"
date: 0001-01-01
description: "本地开发闭环、单元测试、集成测试与常见问题排查。"
summary: "开发与测试 #  快速迭代循环 #  每次改动后的标准步骤：
 说明：以下 bin/* 命令需要在 Easysearch 安装目录执行。
 # 1. 构建（-x test 跳过测试，加速迭代） ./gradlew build -x test # 2. 卸载旧版插件 bin/easysearch-plugin remove my-plugin # 3. 安装新版 bin/easysearch-plugin install \  file:///$(pwd)/build/distributions/my-plugin-0.1.0.zip # 4. 重启 Easysearch bin/easysearch # 5. 验证加载 curl -s http://localhost:9200/_nodes/plugins | \  python3 -m json.tool | grep -A4 &#39;&#34;my-plugin&#34;&#39; 引入 test-framework 依赖 #  Easysearch 官方提供了 test-framework 包，内置随机化测试、嵌入式节点、Hamcrest 断言等工具，所有测试基类均来自这个包。
在插件项目中添加依赖 #  若你的插件项目与 Easysearch 源码在同一 Gradle 多项目构建中，使用项目引用："
---


# 开发与测试

## 快速迭代循环

每次改动后的标准步骤：

> 说明：以下 `bin/*` 命令需要在 Easysearch 安装目录执行。

```bash
# 1. 构建（-x test 跳过测试，加速迭代）
./gradlew build -x test

# 2. 卸载旧版插件
bin/easysearch-plugin remove my-plugin

# 3. 安装新版
bin/easysearch-plugin install \
  file:///$(pwd)/build/distributions/my-plugin-0.1.0.zip

# 4. 重启 Easysearch
bin/easysearch

# 5. 验证加载
curl -s http://localhost:9200/_nodes/plugins | \
  python3 -m json.tool | grep -A4 '"my-plugin"'
```

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

### 测试基类层次

| 基类 | 适用场景 |
|---|---|
| `EasySearchTestCase` | 纯单元测试，无需节点 |
| `ESSingleNodeTestCase` | 单节点集成测试，插件加载验证 |
| `ESIntegTestCase` | 多节点集群集成测试 |

---

## 单元测试（EasySearchTestCase）

继承 `EasySearchTestCase` 可获得随机化测试、内置断言工具等能力，无需启动 Easysearch 节点：

```java
import org.easysearch.test.EasySearchTestCase;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.apache.lucene.analysis.core.WhitespaceTokenizer;

import java.io.StringReader;
import java.util.Set;

public class MyTokenFilterTests extends EasySearchTestCase {

    public void testFilterRemovesStopword() throws Exception {
        // randomAlphaOfLength、randomIntBetween 等随机化工具均由基类提供
        String stopword = randomAlphaOfLengthBetween(3, 8);
        String kept = randomAlphaOfLengthBetween(3, 8);

        Tokenizer tokenizer = new WhitespaceTokenizer();
        tokenizer.setReader(new StringReader(kept + " " + stopword));

        TokenStream stream = new MyStopwordFilter(tokenizer, Set.of(stopword));
        CharTermAttribute term = stream.addAttribute(CharTermAttribute.class);

        stream.reset();
        assertTrue(stream.incrementToken());
        assertEquals(kept, term.toString());
        assertFalse(stream.incrementToken()); // stopword 已被过滤
        stream.end();
        stream.close();
    }

    public void testThrowsOnNullInput() {
        // expectThrows 断言异常类型与消息
        IllegalArgumentException ex = expectThrows(
            IllegalArgumentException.class,
            () -> new MyStopwordFilter(null, Set.of())
        );
        assertTrue(ex.getMessage().contains("tokenizer"));
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

---

## 集成测试（ESSingleNodeTestCase）

通过 `ESSingleNodeTestCase` 启动嵌入式单节点，验证插件在真实节点中的行为。测试类之间共享同一个节点实例，每个测试方法结束后自动清理索引。

```java
import org.easysearch.action.admin.indices.analyze.AnalyzeAction;
import org.easysearch.common.settings.Settings;
import org.easysearch.index.query.QueryBuilders;
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

    public void testAnalyzerRegistered() {
        // createIndex、client() 等工具方法由基类提供
        createIndex("test", Settings.builder()
            .put("index.analysis.analyzer.my_analyzer.type", "custom")
            .put("index.analysis.analyzer.my_analyzer.tokenizer", "standard")
            .put("index.analysis.analyzer.my_analyzer.filter", "my_stopword")
            .build());

        AnalyzeAction.Response response = client()
            .admin().indices()
            .analyze(new AnalyzeAction.Request("test")
                .analyzer("my_analyzer")
                .text("hello world"))
            .actionGet();

        assertEquals(1, response.getTokens().size());
        assertEquals("hello", response.getTokens().get(0).getTerm());
    }

    public void testIndexAndSearch() {
        createIndex("products");
        client().prepareIndex("products")
            .setSource("name", "Easysearch", "category", "search")
            .get();
        client().admin().indices().prepareRefresh("products").get();

        long count = client().prepareSearch("products")
            .setQuery(QueryBuilders.matchQuery("name", "Easysearch"))
            .get()
            .getHits()
            .getTotalHits()
            .value;
        assertEquals(1L, count);
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
./gradlew integTest
./gradlew integTest --tests "com.example.myplugin.MyPluginIntegrationTests"
```

---

## 多节点集成测试（ESIntegTestCase）

`ESIntegTestCase` 会启动一个完整的内嵌集群（默认 1–3 个节点），适合验证分片分配、副本同步等集群级行为。测试类名以 `IT` 结尾时，`integTest` 任务会自动收集。

```java
import org.easysearch.action.admin.cluster.node.info.NodesInfoResponse;
import org.easysearch.action.admin.cluster.node.info.PluginsAndModules;
import org.easysearch.common.settings.Settings;
import org.easysearch.index.query.QueryBuilders;
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

    public void testDataReplicatedAcrossNodes() {
        createIndex("test", Settings.builder()
            .put("index.number_of_shards", 1)
            .put("index.number_of_replicas", 1)
            .build());
        ensureGreen("test");

        indexRandom(true,
            client().prepareIndex("test").setSource("field", randomAlphaOfLength(10)),
            client().prepareIndex("test").setSource("field", randomAlphaOfLength(10))
        );

        long total = client().prepareSearch("test")
            .setQuery(QueryBuilders.matchAllQuery())
            .get().getHits().getTotalHits().value;
        assertEquals(2L, total);
    }
}
```

运行：

```bash
./gradlew integTest
./gradlew integTest --tests "com.example.myplugin.MyPluginClusterIT"
```

## YAML REST 测试

在 `src/yamlRestTest/resources/rest-api-spec/test/` 下写 YAML 测试，验证 HTTP 接口行为；
若使用自定义 API 名称（如 `my_plugin.status`），需先在 `rest-api-spec/api` 定义对应 API：

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

## 验收标准

在提交或发布前确保：

1. `./gradlew test` 全部通过
2. `./gradlew integTest` 全部通过
3. `_nodes/plugins` 能看到插件信息
4. 核心功能有至少一条可复现的 curl 请求

## 下一步

- [插件发布与分享]({{< relref "./publishing.md" >}})


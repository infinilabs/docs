---
title: "结构与类型"
date: 0001-01-01
description: "插件项目结构、关键配置与常见类型的代码示例。"
summary: "结构与类型 #  本页聚焦“结构与选型参考”。如果你要先跑通一个可安装的最小插件，请先看： 插件开发入门。
项目结构 #  my-plugin/ ├── build.gradle # 构建配置 ├── settings.gradle # 项目名称 └── src/ ├── main/java/com/example/myplugin/ │ ├── MyPlugin.java # 插件入口 │ └── analysis/ │ └── MyTokenFilterFactory.java └── test/java/com/example/myplugin/ └── MyPluginTests.java 关键文件 #  plugin-descriptor.properties（构建生成） #  该文件由 Gradle 插件在构建时生成，通常不需要在插件项目里手写；加载时仍会对字段做校验。
name=my-plugin description=My Easysearch Plugin version=0.1.0 easysearch.version=8.0.0 java.version=11 classname=com.example.myplugin.MyPlugin    字段 说明     name 插件唯一标识，用于安装和卸载命令（来源于 esplugin.name）   classname 插件入口类全限定名，必须与代码一致   easysearch."
---


# 结构与类型

本页聚焦“结构与选型参考”。如果你要先跑通一个可安装的最小插件，请先看：[插件开发入门]({{< relref "./getting-started.md" >}})。

## 项目结构

```text
my-plugin/
├── build.gradle                      # 构建配置
├── settings.gradle                   # 项目名称
└── src/
    ├── main/java/com/example/myplugin/
    │   ├── MyPlugin.java             # 插件入口
    │   └── analysis/
    │       └── MyTokenFilterFactory.java
    └── test/java/com/example/myplugin/
        └── MyPluginTests.java
```

## 关键文件

### `plugin-descriptor.properties`（构建生成）

该文件由 Gradle 插件在构建时生成，通常不需要在插件项目里手写；加载时仍会对字段做校验。

```properties
name=my-plugin
description=My Easysearch Plugin
version=0.1.0
easysearch.version=8.0.0
java.version=11
classname=com.example.myplugin.MyPlugin
```

| 字段                   | 说明 |
|----------------------|---|
| `name`               | 插件唯一标识，用于安装和卸载命令（来源于 `esplugin.name`） |
| `classname`          | 插件入口类全限定名，必须与代码一致 |
| `easysearch.version` | 目标 Easysearch 版本，加载时不匹配会拒绝加载 |
| `java.version`       | 插件所需的最低 JDK 版本 |

说明：`easysearch.version` 是描述文件字段；`easysearch_version` 是 `build.gradle` 里的变量名，两者含义不同。

### `build.gradle`
完整 `build.gradle` 示例（含 `buildscript`、仓库与依赖）见：[插件开发入门]({{< relref "./getting-started.md" >}}) 的“配置 `build.gradle`（插件元信息）”。

## 常见插件类型

### 分析器插件（AnalysisPlugin）

最常见的扩展类型，用于自定义分词器、Token 过滤器、字符过滤器。

```java
import org.easysearch.plugins.Plugin;
import org.easysearch.plugins.AnalysisPlugin;
import org.easysearch.index.analysis.TokenFilterFactory;
import org.easysearch.index.analysis.TokenizerFactory;
import org.easysearch.indices.analysis.AnalysisModule.AnalysisProvider;

public class MyPlugin extends Plugin implements AnalysisPlugin {

    @Override
    public Map<String, AnalysisProvider<TokenFilterFactory>> getTokenFilters() {
        return Map.of("my_stopword", MyStopwordFilterFactory::new);
    }

    @Override
    public Map<String, AnalysisProvider<TokenizerFactory>> getTokenizers() {
        return Map.of("my_tokenizer", MyTokenizerFactory::new);
    }
}
```

注册后可以在索引 settings 里直接使用：

```bash
PUT /my-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_filter": { "type": "my_stopword" }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": ["my_filter"]
        }
      }
    }
  }
}
```

### REST/API 插件（ActionPlugin）

用于新增自定义 REST 接口：

```java
import org.easysearch.plugins.Plugin;
import org.easysearch.plugins.ActionPlugin;
import org.easysearch.rest.BaseRestHandler;
import org.easysearch.rest.RestHandler;
import org.easysearch.rest.RestRequest;
import org.easysearch.rest.RestStatus;
import org.easysearch.rest.BytesRestResponse;
import org.easysearch.rest.RestChannelConsumer;
import org.easysearch.client.node.NodeClient;
import org.easysearch.cluster.metadata.IndexNameExpressionResolver;
import org.easysearch.cluster.node.DiscoveryNodes;
import org.easysearch.cluster.service.ClusterService;
import org.easysearch.common.settings.*;

import java.util.List;
import java.util.function.Supplier;

import static org.easysearch.rest.RestRequest.Method.GET;

public class MyPlugin extends Plugin implements ActionPlugin {

    @Override
    public List<RestHandler> getRestHandlers(
            Settings settings,
            RestController restController,
            ClusterSettings clusterSettings,
            IndexScopedSettings indexScopedSettings,
            SettingsFilter settingsFilter,
            IndexNameExpressionResolver indexNameExpressionResolver,
            Supplier<DiscoveryNodes> nodesInCluster) {
        return List.of(new MyRestHandler());
    }
}

// REST 处理器
public class MyRestHandler extends BaseRestHandler {

    @Override
    public List<Route> routes() {
        return List.of(new Route(GET, "/_my_plugin/status"));
    }

    @Override
    public String getName() { return "my_plugin_status"; }

    @Override
    protected RestChannelConsumer prepareRequest(RestRequest request, NodeClient client) {
        return channel -> channel.sendResponse(
            new BytesRestResponse(RestStatus.OK,
                "{\"status\":\"ok\",\"plugin\":\"my-plugin\"}")
        );
    }
}
```

验证：

```bash
curl -s http://localhost:9200/_my_plugin/status
# {"status":"ok","plugin":"my-plugin"}
```

### Ingest 插件（IngestPlugin）

用于在文档写入前做字段转换：

```java
import org.easysearch.plugins.Plugin;
import org.easysearch.plugins.IngestPlugin;
import org.easysearch.ingest.AbstractProcessor;
import org.easysearch.ingest.IngestDocument;
import org.easysearch.ingest.Processor;

import java.util.Map;

public class MyPlugin extends Plugin implements IngestPlugin {

    @Override
    public Map<String, Processor.Factory> getProcessors(Processor.Parameters parameters) {
        return Map.of("uppercase_field", new UppercaseProcessor.Factory());
    }
}

public class UppercaseProcessor extends AbstractProcessor {

    private final String field;

    public UppercaseProcessor(String tag, String description, String field) {
        super(tag, description);
        this.field = field;
    }

    @Override
    public IngestDocument execute(IngestDocument doc) {
        String value = doc.getFieldValue(field, String.class);
        if (value != null) {
            doc.setFieldValue(field, value.toUpperCase());
        }
        return doc;
    }

    @Override
    public String getType() { return "uppercase_field"; }

    public static class Factory implements Processor.Factory {
        @Override
        public UppercaseProcessor create(
                Map<String, Processor.Factory> registry,
                String tag, String description,
                Map<String, Object> config) {
            String field = (String) config.remove("field");
            return new UppercaseProcessor(tag, description, field);
        }
    }
}
```

创建 Pipeline 并写入文档：

```bash
PUT /_ingest/pipeline/my-pipeline
{
  "processors": [
    { "uppercase_field": { "field": "title" } }
  ]
}

POST /my-index/_doc?pipeline=my-pipeline
{ "title": "hello world" }

# 结果：title 被转换为 "HELLO WORLD"
```

## 选型建议

1. 先选改动最小、可快速验证的类型
2. 先做单一职责，避免首版功能过多
3. 每新增一项能力，都补一条最小验证请求

## 下一步

- [开发与测试]({{< relref "./dev-and-test.md" >}})
- [插件发布与分享]({{< relref "./publishing.md" >}})


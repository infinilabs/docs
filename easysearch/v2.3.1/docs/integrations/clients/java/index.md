---
title: "Java 客户端集成与示例"
date: 0001-01-01
description: "使用 Easysearch 官方 Java 客户端连接 Easysearch，完成基础 CRUD 与搜索调用。"
summary: "Java 客户端集成与示例 #  Easysearch Java API Client 是 Easysearch 的官方 Java 客户端，提供了强类型、流式构建器风格的 API 接口。新项目推荐使用此客户端。
 全新重构的 2.0.x 版本，更轻量，移除冗余依赖 兼容 Easysearch 各版本 支持阻塞和异步两种调用方式 使用流式构建器和函数式模式，代码简洁易读 通过 Jackson 无缝集成应用类  相关指南 #    官方 Java Client API 文档  如何使用 Curl 操作  依赖引入 #  Maven #  &lt;dependency&gt; &lt;groupId&gt;com.infinilabs&lt;/groupId&gt; &lt;artifactId&gt;easysearch-client&lt;/artifactId&gt; &lt;version&gt;2.0.2&lt;/version&gt; &lt;/dependency&gt; Gradle #  implementation &#39;com.infinilabs:easysearch-client:2.0.2&#39;  已发布到 Maven 中央仓库： mvnrepository.com/artifact/com.infinilabs/easysearch-client，需要 JDK 8 或以上。
 连接配置 #  基础连接（带认证 + HTTPS） #  Easysearch 默认启用安全认证和 HTTPS，以下代码展示完整的连接初始化："
---


# Java 客户端集成与示例

**Easysearch Java API Client** 是 Easysearch 的官方 Java 客户端，提供了强类型、流式构建器风格的 API 接口。新项目推荐使用此客户端。

- 全新重构的 2.0.x 版本，更轻量，移除冗余依赖
- 兼容 Easysearch 各版本
- 支持阻塞和异步两种调用方式
- 使用流式构建器和函数式模式，代码简洁易读
- 通过 Jackson 无缝集成应用类

## 相关指南

- [官方 Java Client API 文档](https://docs.infinilabs.com/easysearch/main/docs/references/client/java-client/)
- [如何使用 Curl 操作]({{< relref "../../quick-start/connect/curl.md" >}})

## 依赖引入

### Maven

```xml
<dependency>
  <groupId>com.infinilabs</groupId>
  <artifactId>easysearch-client</artifactId>
  <version>2.0.2</version>
</dependency>
```

### Gradle

```groovy
implementation 'com.infinilabs:easysearch-client:2.0.2'
```

> 已发布到 Maven 中央仓库：[mvnrepository.com/artifact/com.infinilabs/easysearch-client](https://mvnrepository.com/artifact/com.infinilabs/easysearch-client/2.0.2)，需要 JDK 8 或以上。

## 连接配置

### 基础连接（带认证 + HTTPS）

Easysearch 默认启用安全认证和 HTTPS，以下代码展示完整的连接初始化：

```java
import com.infinilabs.clients.easysearch.EasysearchClient;
import com.infinilabs.clients.json.jackson.JacksonJsonpMapper;
import com.infinilabs.clients.transport.rest_client.RestClientTransport;
import com.infinilabs.clients.transport.EasysearchTransport;

import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.apache.http.nio.conn.ssl.SSLIOSessionStrategy;
import org.elasticsearch.client.RestClient;

import javax.net.ssl.SSLContext;
import org.apache.http.ssl.SSLContextBuilder;

public static EasysearchClient create() throws Exception {
    HttpHost[] hosts = new HttpHost[]{new HttpHost("localhost", 9200, "https")};

    // SSL 配置（开发环境信任所有证书）
    SSLContext sslContext = SSLContextBuilder.create()
        .loadTrustMaterial(null, (chains, authType) -> true)
        .build();
    SSLIOSessionStrategy sessionStrategy =
        new SSLIOSessionStrategy(sslContext, NoopHostnameVerifier.INSTANCE);

    // 认证
    BasicCredentialsProvider credsProv = new BasicCredentialsProvider();
    credsProv.setCredentials(AuthScope.ANY,
        new UsernamePasswordCredentials("admin", "your_password"));

    // 创建 RestClient
    RestClient restClient = RestClient.builder(hosts)
        .setHttpClientConfigCallback(httpBuilder ->
            httpBuilder
                .setDefaultCredentialsProvider(credsProv)
                .setSSLStrategy(sessionStrategy)
                .disableAuthCaching()
        )
        .setRequestConfigCallback(reqBuilder ->
            reqBuilder.setConnectTimeout(30000).setSocketTimeout(300000)
        )
        .build();

    // 创建 EasysearchClient
    EasysearchTransport transport = new RestClientTransport(
        restClient, new JacksonJsonpMapper());
    return new EasysearchClient(transport);
}
```

> **生产环境建议**：使用正规 CA 签发的证书或导入 Easysearch 的 CA 证书到信任库中，而非信任所有证书。

## 基础 CRUD 操作

### 索引文档

```java
EasysearchClient client = create();

// 使用 Map
Map<String, Object> doc = Map.of(
    "name", "Easysearch",
    "category", "database"
);
client.index(i -> i.index("products").id("1").document(doc));
```

### 获取文档

```java
var response = client.get(g -> g.index("products").id("1"), Map.class);
if (response.found()) {
    System.out.println(response.source());
}
```

### 删除文档

```java
client.delete(d -> d.index("products").id("1"));
```

## 搜索查询

```java
var response = client.search(s -> s
    .index("products")
    .query(q -> q.match(m -> m.field("name").query("Easysearch")))
    .from(0)
    .size(10),
    Map.class
);

response.hits().hits().forEach(hit ->
    System.out.println(hit.source())
);
```

## 批量操作

```java
import com.infinilabs.clients.easysearch.core.bulk.BulkRequest;

var result = client.bulk(b -> b
    .operations(o -> o.index(i -> i.index("products").id("2")
        .document(Map.of("name", "Console", "category", "tool"))))
    .operations(o -> o.index(i -> i.index("products").id("3")
        .document(Map.of("name", "Gateway", "category", "proxy"))))
);

if (result.errors()) {
    result.items().stream()
        .filter(item -> item.error() != null)
        .forEach(item -> System.err.println(item.error().reason()));
}
```

## 生产环境配置要点

| 配置项           | 推荐值                | 说明                                          |
| ---------------- | --------------------- | --------------------------------------------- |
| 连接超时         | 5000~30000ms          | `setConnectTimeout()`                         |
| Socket 超时      | 60000~300000ms        | `setSocketTimeout()`                          |
| 最大连接数       | 30~100                | `setMaxConnTotal(100)`                        |
| 多节点负载均衡   | 配置所有节点          | `RestClient.builder(host1, host2, host3)`     |
| 关闭客户端       | 应用退出时调用        | `restClient.close()`                          |

> **提示**：在微服务架构中，建议将 `EasysearchClient` 作为单例 Bean 管理（如 Spring `@Bean`），避免频繁创建和销毁连接。

## 已有项目兼容说明

如果你有已有项目使用 Elasticsearch 7.10 的 `RestHighLevelClient`，也可以直接连接 Easysearch（需在 `easysearch.yml` 中开启 `elasticsearch.api_compatibility: true`）。但**新项目推荐使用 Easysearch 官方客户端**，获得更好的类型安全和 API 体验。



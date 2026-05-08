---
title: "使用 Java Client 连接 Easysearch"
date: 0001-01-01
description: "Easysearch 官方 Java 客户端快速入门，完成连接、CRUD 与搜索调用。"
summary: "Java 客户端快速入门 #  本页面帮助你快速跑通 Easysearch Java API Client 连接 Easysearch 的完整流程。更深入的用法请参阅 Java 客户端详细指南。
Easysearch Java API Client 是 Easysearch 的官方 Java 客户端，提供了强类型、流式构建器风格的 API 接口：
 全新 2.0.x 版本，更轻量，移除冗余依赖 兼容 Easysearch 各版本 支持阻塞和异步两种调用方式 使用 Jackson 无缝集成应用类  添加依赖 #  Maven：
&lt;dependency&gt; &lt;groupId&gt;com.infinilabs&lt;/groupId&gt; &lt;artifactId&gt;easysearch-client&lt;/artifactId&gt; &lt;version&gt;2.0.2&lt;/version&gt; &lt;/dependency&gt; Gradle：
implementation &#39;com.infinilabs:easysearch-client:2.0.2&#39;  💡 已发布到 Maven 中央仓库： mvnrepository.com/artifact/com.infinilabs/easysearch-client，需要 JDK 8 或以上版本。
 建立连接 #  import com.infinilabs.clients.easysearch.EasysearchClient; import com.infinilabs.clients.json.jackson.JacksonJsonpMapper; import com.infinilabs.clients.transport.rest_client.RestClientTransport; import com.infinilabs.clients.transport.EasysearchTransport; import org."
---


# Java 客户端快速入门

本页面帮助你快速跑通 **Easysearch Java API Client** 连接 Easysearch 的完整流程。更深入的用法请参阅 [Java 客户端详细指南]({{< relref "/docs/integrations/clients/java.md" >}})。

Easysearch Java API Client 是 Easysearch 的**官方 Java 客户端**，提供了强类型、流式构建器风格的 API 接口：

- 全新 2.0.x 版本，更轻量，移除冗余依赖
- 兼容 Easysearch 各版本
- 支持阻塞和异步两种调用方式
- 使用 Jackson 无缝集成应用类

## 添加依赖

**Maven**：

```xml
<dependency>
  <groupId>com.infinilabs</groupId>
  <artifactId>easysearch-client</artifactId>
  <version>2.0.2</version>
</dependency>
```

**Gradle**：

```groovy
implementation 'com.infinilabs:easysearch-client:2.0.2'
```

> 💡 已发布到 Maven 中央仓库：[mvnrepository.com/artifact/com.infinilabs/easysearch-client](https://mvnrepository.com/artifact/com.infinilabs/easysearch-client/2.0.2)，需要 JDK 8 或以上版本。

## 建立连接

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

    // SSL 配置（开发环境信任所有证书，生产环境请配置 CA 证书）
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

## 索引文档

```java
EasysearchClient client = create();

// 使用 Map
Map<String, Object> doc = Map.of(
    "title", "Easysearch 入门",
    "views", 100
);
client.index(i -> i.index("articles").id("1").document(doc));
```

## 搜索

```java
var response = client.search(s -> s
    .index("articles")
    .query(q -> q.match(m -> m.field("title").query("Easysearch"))),
    Map.class
);

response.hits().hits().forEach(hit ->
    System.out.println(hit.source())
);
```

## 关闭连接

```java
// 关闭底层 RestClient
restClient.close();
```

## 注意事项

| 事项 | 说明 |
|------|------|
| 客户端 | 使用官方 `com.infinilabs:easysearch-client:2.0.2` |
| JDK 版本 | 需要 JDK 8 或以上 |
| 证书 | 生产环境应配置 CA 证书，而非信任所有证书 |

> 💡 如果你有已有项目使用 Elasticsearch 7.10 的 `RestHighLevelClient`，也可以兼容连接 Easysearch（需开启 `elasticsearch.api_compatibility: true`）。但新项目推荐使用 Easysearch 官方客户端。

## 相关文档

- [Java 客户端完整指南]({{< relref "/docs/integrations/clients/java.md" >}})：连接池、批量操作、生产配置
- [官方 Java Client API 文档](https://docs.infinilabs.com/easysearch/main/docs/references/client/java-client/)
- [使用 Curl 访问 Easysearch]({{< relref "./curl.md" >}})
- [入门教程]({{< relref "../tutorial.md" >}})



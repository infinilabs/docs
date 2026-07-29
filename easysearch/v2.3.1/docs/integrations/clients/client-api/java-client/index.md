---
title: "Java 客户端"
date: 0001-01-01
summary: "Java 客户端 #  Easysearch Java API Client 是 Easysearch 的官方 Java 客户端，提供了简洁、强大且类型安全的 API 接口。
相关指南（先读这些） #     Java 客户端集成
   客户端集成
  全新重构的 2.0.x 版本，更轻量级的设计，移除冗余依赖。
  兼容 Easysearch 各个版本。
  为常用 Easysearch API 提供强类型的请求和响应。
  API 均支持阻塞和异步方式。
  使用流式构建器和函数式模式，以在创建复杂嵌套结构时编写简洁易读的代码。
  通过使用 Jackson 无缝集成应用程序类。
  快速开始 #  本页指导您完成Java客户端的安装过程，展示了如何实例化客户端，以及如何使用它执行基本的 Easysearch 操作。
安装 #  easysearch-client 已经发布到 Maven https://mvnrepository.com/artifact/com.infinilabs/easysearch-client/2.0.2"
---


# Java 客户端

Easysearch Java API Client 是 Easysearch 的官方 Java 客户端，提供了简洁、强大且类型安全的 API 接口。

## 相关指南（先读这些）

- [Java 客户端集成]({{< relref "/docs/integrations/clients/java.md" >}})
- [客户端集成]({{< relref "/docs/integrations/clients/client-api/_index.md" >}})

- 全新重构的 2.0.x 版本，更轻量级的设计，移除冗余依赖。
- 兼容 Easysearch 各个版本。
- 为常用 Easysearch API 提供强类型的请求和响应。
- API 均支持阻塞和异步方式。
- 使用流式构建器和函数式模式，以在创建复杂嵌套结构时编写简洁易读的代码。
- 通过使用 Jackson 无缝集成应用程序类。

## 快速开始

本页指导您完成Java客户端的安装过程，展示了如何实例化客户端，以及如何使用它执行基本的 Easysearch 操作。

### 安装

easysearch-client 已经发布到 Maven https://mvnrepository.com/artifact/com.infinilabs/easysearch-client/2.0.2

安装需要 jdk8 或以上版本

easysearch-client 使用 Jackson 将业务代码和客户端 api 进行集成。

#### 在 Maven 项目中安装

相比 1.x 版本的客户端，新版客户端的安装更加简单，只需在您项目的 pom 文件的 dependencies 区域添加以下依赖以引入客户端

```xml

<dependencies>
    <dependency>
        <groupId>com.infinilabs</groupId>
        <artifactId>easysearch-client</artifactId>
        <version>2.0.2</version>
    </dependency>
</dependencies>

```

#### 在 gradle 项目中安装

在您项目的 build.gradle 文件的 dependencies 区域添加以下依赖
```gradle
dependencies {
    implementation 'com.infinilabs:easysearch-client:2.0.2'
}
```

### 初始化客户端

假设您本地启动了 Easysearch 服务，并启用了安全通信加密和 security， 可以使用以下代码初始化客户端连接。

```java
 public static EasysearchClient create() throws NoSuchAlgorithmException, KeyStoreException,
        KeyManagementException {
        boolean https = true;

        final HttpHost[] hosts = new HttpHost[]{new HttpHost("localhost", 9200, "https")};

        final SSLContext sslContext = SSLContextBuilder.create()
            .loadTrustMaterial(null, (chains, authType) -> true).build();
        SSLIOSessionStrategy sessionStrategy = new SSLIOSessionStrategy(sslContext, NoopHostnameVerifier.INSTANCE);

        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials("username", "passwowd"));

        RestClient restClient = RestClient.builder(hosts)
            .setHttpClientConfigCallback(httpClientBuilder ->
                httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider)
                    .setSSLStrategy(sessionStrategy)
                    .disableAuthCaching()
            ).setRequestConfigCallback(requestConfigCallback -> 
            requestConfigCallback.setConnectTimeout(30000).setSocketTimeout(300000))
            .build();

        EasysearchTransport transport = new RestClientTransport(
            restClient, new JacksonJsonpMapper());
        return new EasysearchClient(transport);
    }

```


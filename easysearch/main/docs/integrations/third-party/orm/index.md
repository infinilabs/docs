---
title: "ORM 框架集成"
date: 0001-01-01
description: "MyBatis、Spring Data 等 ORM 与 Easysearch 集成。"
summary: "ORM 框架集成 #  在使用关系数据库的应用中，常见需求是将数据库数据同步到 Easysearch 以实现搜索功能。本文介绍常见 ORM 框架与 Easysearch 的集成方式。
架构模式 #  模式一：应用层双写 #  应用程序 → 写入 MySQL (ORM) → 同时写入 Easysearch (ES Client) 优点：实现简单，实时性好 缺点：一致性难保障，应用耦合度高
模式二：CDC 同步（推荐） #  应用程序 → 写入 MySQL ↓ Canal / Debezium 监听 binlog ↓ 同步到 Easysearch 优点：应用无感知，一致性好 缺点：需要额外组件
模式三：定时全量/增量同步 #  定时任务 → 查询 MySQL 增量数据 → 批量写入 Easysearch 优点：简单可靠 缺点：有延迟（取决于同步周期）
Spring Data Elasticsearch #  Spring Data 提供了对 Elasticsearch 的原生支持，可直接连接 Easysearch："
---


# ORM 框架集成

在使用关系数据库的应用中，常见需求是将数据库数据同步到 Easysearch 以实现搜索功能。本文介绍常见 ORM 框架与 Easysearch 的集成方式。

## 架构模式

### 模式一：应用层双写

```
应用程序 → 写入 MySQL (ORM)
        → 同时写入 Easysearch (ES Client)
```

优点：实现简单，实时性好
缺点：一致性难保障，应用耦合度高

### 模式二：CDC 同步（推荐）

```
应用程序 → 写入 MySQL
                ↓
        Canal / Debezium 监听 binlog
                ↓
        同步到 Easysearch
```

优点：应用无感知，一致性好
缺点：需要额外组件

### 模式三：定时全量/增量同步

```
定时任务 → 查询 MySQL 增量数据
        → 批量写入 Easysearch
```

优点：简单可靠
缺点：有延迟（取决于同步周期）

## Spring Data Elasticsearch

Spring Data 提供了对 Elasticsearch 的原生支持，可直接连接 Easysearch：

### 1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

> 使用 Spring Boot 2.x 系列（对应 ES 7.x 客户端）。

### 2. 配置连接

```yaml
# application.yml
spring:
  elasticsearch:
    uris: https://localhost:9200
    username: admin
    password: your-password
```

SSL 配置（自签名证书）：

```java
@Configuration
public class EsConfig extends AbstractElasticsearchConfiguration {
    @Override
    public RestHighLevelClient elasticsearchClient() {
        RestClientBuilder builder = RestClient.builder(
            new HttpHost("localhost", 9200, "https")
        ).setHttpClientConfigCallback(httpClientBuilder ->
            httpClientBuilder
                .setDefaultCredentialsProvider(credentialsProvider())
                .setSSLContext(sslContext())  // 信任自签名证书
        );
        return new RestHighLevelClient(builder);
    }
}
```

### 3. 定义实体

```java
@Document(indexName = "product")
public class Product {
    @Id
    private String id;

    @Field(type = FieldType.Text, analyzer = "ik_smart")
    private String title;

    @Field(type = FieldType.Keyword)
    private String category;

    @Field(type = FieldType.Double)
    private Double price;
}
```

### 4. 定义 Repository

```java
public interface ProductRepository extends ElasticsearchRepository<Product, String> {

    List<Product> findByTitleContaining(String keyword);

    List<Product> findByCategoryAndPriceLessThan(String category, Double price);
}
```

## MyBatis + ES Client 双写

```java
@Service
@Transactional
public class ProductService {

    @Autowired
    private ProductMapper mysqlMapper;  // MyBatis Mapper

    @Autowired
    private RestHighLevelClient esClient;

    public void createProduct(Product product) {
        // 1. 写入 MySQL
        mysqlMapper.insert(product);

        // 2. 同步到 Easysearch
        IndexRequest request = new IndexRequest("product")
            .id(product.getId())
            .source(JSON.toJSONString(product), XContentType.JSON);
        esClient.index(request, RequestOptions.DEFAULT);
    }
}
```

> 双写模式下，建议通过消息队列解耦，避免 ES 写入失败影响主流程。

## Canal 数据同步

[Canal](https://github.com/alibaba/canal) 可以监听 MySQL binlog，将变更实时同步到 Easysearch：

```
MySQL binlog → Canal Server → Canal Client → Easysearch
```

Canal Adapter 内置了 ES 适配器，配置示例：

```yaml
# application.yml (canal-adapter)
canalAdapters:
  - instance: example
    groups:
      - outAdapters:
          - name: es7
            hosts: https://localhost:9200
            properties:
              security.auth: admin:your-password
              cluster.name: easysearch
```

## 注意事项

| 注意项 | 说明 |
|--------|------|
| API 兼容 | 开启 `elasticsearch.api_compatibility: true` |
| 客户端版本 | 使用 ES 7.10.2 oss 版本的客户端 |
| HTTPS | 需配置 SSL 信任 |
| 一致性 | 双写模式考虑最终一致性方案 |
| 事务 | Easysearch 不支持分布式事务，搜索数据允许短暂延迟 |

## 延伸阅读

- [Java 客户端]({{< relref "/docs/integrations/clients/java.md" >}})
- [Easy-ES 查询框架]({{< relref "./easy-es.md" >}})
- [数据接入]({{< relref "/docs/integrations/ingest/" >}})
- [SeaTunnel 集成]({{< relref "./seatunnel.md" >}})


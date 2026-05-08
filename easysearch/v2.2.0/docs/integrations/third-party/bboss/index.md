---
title: "Bboss 集成"
date: 0001-01-01
description: "Bboss 与 Easysearch 原生集成，一个高性能高兼容性的 Elasticsearch/Easysearch/Opensearch Java 客户端框架"
summary: "Bboss 集成 #   Bboss 是一款高性能、高兼容性的搜索引擎 Java REST 客户端框架，基于 Apache License 2.0 开源，原生支持 Elasticsearch、Easysearch 和 Opensearch。自带客户端集群节点负载均衡和容灾，多集群多数据源，自动索引托管，多种分页机制，傻瓜级 CRUD，脚本，SQL，JDBC，高亮，权重，聚合，IP，GEO 地理位置，父子嵌套，应有尽有。
 源码仓库： https://gitee.com/bboss/bboss-elastic 官方文档： https://esdoc.bbossgroups.com/ Maven 中央仓库： bboss-datatran-jdbc   核心优势 #     特性 说明     学习成本低 无需学习额外 API，只需掌握 Elasticsearch DSL，极简使用方式   原生多引擎支持 完美支持 ES 1.x ~ 9.x、Easysearch 1.x ~ 2.x+、Opensearch 1.x ~ 2.x+   开箱即用 Spring Boot 自动配置，无需复杂设置   高效异步处理 内置 BulkProcessor 异步批处理器，大幅提升写入性能   灵活查询方式 支持 DSL、SQL、O/R Mapping 多种查询模式   多数据源支持 一个应用可同时操作多个不同版本的搜索引擎集群   完整的结果封装 返回结果支持 JSON、PO 对象、List 集合、Map 等多种类型   客户端负载均衡 默认启用客户端负载均衡，容灾性更好     快速开始 #  1."
---


# Bboss 集成

[Bboss](https://esdoc.bbossgroups.com/) 是一款高性能、高兼容性的搜索引擎 Java REST 客户端框架，基于 Apache License 2.0 开源，原生支持 Elasticsearch、Easysearch 和 Opensearch。自带客户端集群节点负载均衡和容灾，多集群多数据源，自动索引托管，多种分页机制，傻瓜级 CRUD，脚本，SQL，JDBC，高亮，权重，聚合，IP，GEO 地理位置，父子嵌套，应有尽有。

![Bboss 集成方案](https://esdoc.bbossgroups.com/images/client-Elasticsearch.png)

- 源码仓库：[https://gitee.com/bboss/bboss-elastic](https://gitee.com/bboss/bboss-elastic)
- 官方文档：[https://esdoc.bbossgroups.com/](https://esdoc.bbossgroups.com/)
- Maven 中央仓库：[bboss-datatran-jdbc](https://mvnrepository.com/artifact/com.bbossgroups.plugins/bboss-datatran-jdbc)

---

## 核心优势

| 特性               | 说明                                                                |
| ------------------ | ------------------------------------------------------------------- |
| **学习成本低**     | 无需学习额外 API，只需掌握 Elasticsearch DSL，极简使用方式          |
| **原生多引擎支持** | 完美支持 ES 1.x ~ 9.x、Easysearch 1.x ~ 2.x+、Opensearch 1.x ~ 2.x+ |
| **开箱即用**       | Spring Boot 自动配置，无需复杂设置                                  |
| **高效异步处理**   | 内置 BulkProcessor 异步批处理器，大幅提升写入性能                   |
| **灵活查询方式**   | 支持 DSL、SQL、O/R Mapping 多种查询模式                             |
| **多数据源支持**   | 一个应用可同时操作多个不同版本的搜索引擎集群                        |
| **完整的结果封装** | 返回结果支持 JSON、PO 对象、List 集合、Map 等多种类型               |
| **客户端负载均衡** | 默认启用客户端负载均衡，容灾性更好                                  |

---

## 快速开始

### 1. 添加依赖

#### Maven

根据 Spring Boot 版本选择对应的 starter：

**Spring Boot 1.x / 2.x：**

```xml
<dependencies>
    <!-- Bboss 核心依赖 -->
    <dependency>
        <groupId>com.bbossgroups.plugins</groupId>
        <artifactId>bboss-datatran-jdbc</artifactId>
        <version>7.5.5</version>
    </dependency>

    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>com.bbossgroups.plugins</groupId>
        <artifactId>bboss-elasticsearch-spring-boot-starter</artifactId>
        <version>7.5.5</version>
    </dependency>

    <!-- 其他依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**Spring Boot 3.x+：**

```xml
<dependencies>
    <dependency>
        <groupId>com.bbossgroups.plugins</groupId>
        <artifactId>bboss-datatran-jdbc</artifactId>
        <version>7.5.5</version>
    </dependency>

    <!-- Spring Boot 3.x Starter -->
    <dependency>
        <groupId>com.bbossgroups.plugins</groupId>
        <artifactId>bboss-elasticsearch-spring-boot3-starter</artifactId>
        <version>7.5.5</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

#### Gradle

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.0'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
}

dependencies {
    implementation 'com.bbossgroups.plugins:bboss-datatran-jdbc:7.5.5'
    implementation 'com.bbossgroups.plugins:bboss-elasticsearch-spring-boot-starter:7.5.5'
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

### 2. 配置连接

在 `application.yml` 或 `application.properties` 中配置 Easysearch 连接信息：

#### YAML 格式

```yaml
spring:
  elasticsearch:
    bboss:
      elasticsearch:
        rest:
          hostNames: localhost:9200
        # 启用 HTTPS（Easysearch 默认启用）
        useHttps: true
        # restHttp: https://
      # 认证信息
      elasticUser: admin
      elasticPassword: your_password_here
```

#### Properties 格式

```properties
spring.elasticsearch.bboss.elasticsearch.rest.hostNames=localhost:9200
spring.elasticsearch.bboss.elasticsearch.useHttps=true
spring.elasticsearch.bboss.elasticUser=admin
spring.elasticsearch.bboss.elasticPassword=your_password_here
```

**配置说明**：

| 参数                           | 说明                                   | 示例                                |
| ------------------------------ | -------------------------------------- | ----------------------------------- |
| `elasticsearch.rest.hostNames` | Easysearch 服务地址，多个逗号分隔      | `10.20.30.40:9200,10.20.30.41:9200` |
| `elasticsearch.useHttps`       | 是否使用 HTTPS（Easysearch 默认 true） | `true`                              |
| `elasticUser`                  | 用户名（可选，如启用认证）             | `admin`                             |
| `elasticPassword`              | 密码（可选，如启用认证）               | `your_password_here`                |

### 3. 定义实体类

```java
import org.frameworkset.elasticsearch.entity.ESDatas;

public class Document {

    /** 文档 ID，标识唯一性（对应索引中的 _id） */
    private String id;

    /** 文档标题 */
    private String title;

    /** 文档内容 - 分词字段 */
    private String content;

    /** 作者 */
    private String author;

    /** 创建时间 */
    private Long createTime;

    /** 点赞数 */
    private Integer likes;

    // Getters & Setters
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }

    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }

    public Long getCreateTime() { return createTime; }
    public void setCreateTime(Long createTime) { this.createTime = createTime; }

    public Integer getLikes() { return likes; }
    public void setLikes(Integer likes) { this.likes = likes; }
}
```

### 4. 创建索引并插入数据

```java
import org.frameworkset.elasticsearch.boot.BBossESStarter;
import org.frameworkset.elasticsearch.client.ClientInterface;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.Date;

@Service
public class DocumentService {

    @Autowired
    private BBossESStarter bbossESStarter; // 注入 Bboss 客户端启动器

    /**
     * 插入单个文档
     */
    public void insertDocument() {
        // 获取不加载 DSL 配置文件的客户端（用于基础操作）
        ClientInterface clientUtil = bbossESStarter.getRestClient();

        Document doc = new Document();
        doc.setId("1");
        doc.setTitle("Easysearch 与 Bboss 集成");
        doc.setContent("这是一篇关于 Easysearch 与 Bboss ElasticsearchClient 集成的文章");
        doc.setAuthor("admin");
        doc.setCreateTime(System.currentTimeMillis());
        doc.setLikes(100);

        // 添加文档，如文档 ID 已存在则更新
        String response = clientUtil.addDocument("documents", // 索引名
                                                 doc,          // 文档对象
                                                 "refresh=true"); // 直接刷新
        System.out.println("Insert response: " + response);
    }

    /**
     * 批量插入文档
     */
    public void batchInsertDocuments() {
        ClientInterface clientUtil = bbossESStarter.getRestClient();

        // 构建文档列表
        java.util.List<Document> documents = new java.util.ArrayList<>();
        for (int i = 2; i <= 5; i++) {
            Document doc = new Document();
            doc.setId(String.valueOf(i));
            doc.setTitle("Easysearch 文章 " + i);
            doc.setContent("这是第 " + i + " 篇文章内容");
            doc.setAuthor("author" + i);
            doc.setCreateTime(System.currentTimeMillis());
            doc.setLikes(i * 10);
            documents.add(doc);
        }

        // 批量添加
        String response = clientUtil.addDocuments("documents", documents, "refresh=true");
        System.out.println("Batch insert response: " + response);
    }

    /**
     * 根据 ID 获取文档
     */
    public void getDocumentById() {
        ClientInterface clientUtil = bbossESStarter.getRestClient();

        Document doc = clientUtil.getDocument("documents", // 索引名
                                              "1",          // 文档 ID
                                              Document.class); // 返回类型
        if (doc != null) {
            System.out.println("Title: " + doc.getTitle());
            System.out.println("Content: " + doc.getContent());
        }
    }

    /**
     * 更新文档
     */
    public void updateDocument() {
        ClientInterface clientUtil = bbossESStarter.getRestClient();

        Document doc = new Document();
        doc.setId("1");
        doc.setTitle("更新标题");
        doc.setContent("更新内容");
        doc.setLikes(200);

        String response = clientUtil.addDocument("documents", doc, "refresh=true");
        System.out.println("Update response: " + response);
    }

    /**
     * 删除文档
     */
    public void deleteDocument() {
        ClientInterface clientUtil = bbossESStarter.getRestClient();

        String response = clientUtil.deleteDocument("documents", // 索引名
                                                    "1");         // 文档 ID
        System.out.println("Delete response: " + response);
    }

    /**
     * 批量删除文档
     */
    public void batchDeleteDocuments() {
        ClientInterface clientUtil = bbossESStarter.getRestClient();

        String[] ids = {"2", "3", "4"};
        String response = clientUtil.deleteDocuments("documents", ids);
        System.out.println("Batch delete response: " + response);
    }
}
```

### 5. 查询操作

#### 基础查询

```java
/**
 * 查询所有文档
 */
public void searchAll() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    // 查询所有文档，默认返回 5000 条
    ESDatas<Document> esDatas = clientUtil.searchAll("documents", Document.class);

    // 获取查询结果列表
    java.util.List<Document> documents = esDatas.getDatas();
    System.out.println("Total size: " + esDatas.getTotalSize());
    System.out.println("Found documents: " + documents.size());
}

/**
 * 分页查询所有文档
 */
public void searchAllWithPagination() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    // 使用 ScrollHandler 分页处理数据（适合大数据量）
    clientUtil.searchAll("documents", 1000, // 每页大小
        new org.frameworkset.elasticsearch.scroll.ScrollHandler<Document>() {
            public void handle(ESDatas<Document> esDatas,
                             org.frameworkset.elasticsearch.scroll.HandlerInfo handlerInfo) throws Exception {
                java.util.List<Document> dataList = esDatas.getDatas();
                System.out.println("Current batch size: " + dataList.size());
                System.out.println("Total matched: " + esDatas.getTotalSize());
                // 处理每一批数据
                for (Document doc : dataList) {
                    System.out.println("ID: " + doc.getId() + ", Title: " + doc.getTitle());
                }
            }
        }, Document.class);
}

/**
 * 并行查询所有文档
 */
public void searchAllParallel() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    // 使用 2 个线程并行查询
    ESDatas<Document> esDatas = clientUtil.searchAllParallel("documents", Document.class, 2);
    System.out.println("Total size: " + esDatas.getTotalSize());
    System.out.println("Found documents: " + esDatas.getDatas().size());
}

/**
 * 根据字段值查询
 */
public void searchByField() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    // 按 author 字段查询，返回分页结果
    ESDatas<Document> esDatas = clientUtil.searchListByField("documents",
                                                             "author.keyword",  // 字段名
                                                             "admin",           // 字段值
                                                             Document.class,    // 返回类型
                                                             0,                 // 分页起始位置
                                                             10);               // 每页大小

    System.out.println("Total matched: " + esDatas.getTotalSize());
    for (Document doc : esDatas.getDatas()) {
        System.out.println("ID: " + doc.getId() + ", Title: " + doc.getTitle());
    }
}

/**
 * 统计文档数量
 */
public void countDocuments() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    long count = clientUtil.countAll("documents");
    System.out.println("Total documents: " + count);
}
```

#### DSL 查询（高级）

首先，在 `src/main/resources/esmapper` 目录下创建 DSL 配置文件 `document.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<properties>
    <!-- 复杂搜索查询 -->
    <property name="searchByKeyword">
        <![CDATA[
        {
            "query": {
                "bool": {
                    "must": [
                        {"match": {"content": "#[keyword]"}}
                    ],
                    "filter": {
                        "range": {
                            "createTime": {
                                "gte": #[startTime],
                                "lte": #[endTime]
                            }
                        }
                    }
                }
            },
            "sort": [
                {"createTime": {"order": "desc"}},
                {"likes": {"order": "desc"}}
            ],
            "from": #[from:0],
            "size": #[size:20]
        }
        ]]>
    </property>

    <!-- 高亮搜索 -->
    <property name="searchWithHighlight">
        <![CDATA[
        {
            "query": {
                "match": {"content": "#[keyword]"}
            },
            "highlight": {
                "pre_tags": ["<em>"],
                "post_tags": ["</em>"],
                "fields": {
                    "content": {}
                }
            },
            "size": #[size:20]
        }
        ]]>
    </property>
</properties>
```

使用 DSL 配置进行查询：

```java
/**
 * 使用 DSL 配置查询
 */
public void searchWithDSL() {
    // 创建加载 DSL 配置文件的客户端
    ClientInterface clientUtil = bbossESStarter.getConfigRestClient("esmapper/document.xml");

    // 准备查询参数
    java.util.Map<String, Object> params = new java.util.HashMap<>();
    params.put("keyword", "Easysearch");
    params.put("startTime", System.currentTimeMillis() - 30 * 24 * 3600 * 1000L); // 30 天前
    params.put("endTime", System.currentTimeMillis());
    params.put("from", 0);
    params.put("size", 20);

    // 执行查询
    ESDatas<Document> esDatas = clientUtil.searchList("documents/_search",
                                                      "searchByKeyword",  // DSL 名称
                                                      params,             // 查询参数
                                                      Document.class);    // 返回类型

    System.out.println("Total matched: " + esDatas.getTotalSize());
    for (Document doc : esDatas.getDatas()) {
        System.out.println("ID: " + doc.getId() + ", Title: " + doc.getTitle());
    }
}

/**
 * 高亮搜索
 */
public void highlightSearch() {
    ClientInterface clientUtil = bbossESStarter.getConfigRestClient("esmapper/document.xml");

    java.util.Map<String, Object> params = new java.util.HashMap<>();
    params.put("keyword", "Easysearch");
    params.put("size", 20);

    ESDatas<Document> esDatas = clientUtil.searchList("documents/_search",
                                                      "searchWithHighlight",
                                                      params,
                                                      Document.class);

    System.out.println("Found " + esDatas.getDatas().size() + " documents");
}
```

### 6. 异步批处理

使用 BulkProcessor 进行高效的异步批处理：

```java
import org.frameworkset.elasticsearch.bulk.BulkProcessor;
import org.frameworkset.elasticsearch.bulk.BulkProcsJob;

/**
 * 异步批量插入文档
 */
public void asyncBatchInsert() throws Exception {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    // 创建异步批处理器
    BulkProcessor bulkProcessor = new BulkProcessor(clientUtil, 100, 5000L);

    try {
        // 插入 10000 条文档
        for (int i = 0; i < 10000; i++) {
            Document doc = new Document();
            doc.setId(String.valueOf(i));
            doc.setTitle("Document " + i);
            doc.setContent("Content for document " + i);
            doc.setAuthor("author");
            doc.setCreateTime(System.currentTimeMillis());
            doc.setLikes(i);

            // 添加到批处理器
            bulkProcessor.insertDocument("documents", doc);
        }

        // 等待所有请求完成
        bulkProcessor.shutdown();
        System.out.println("Async batch insert completed");

    } catch (Exception e) {
        e.printStackTrace();
        bulkProcessor.shutdown();
    }
}

/**
 * 监听批处理进度
 */
public void monitorBulkProcessing() throws Exception {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    BulkProcessor bulkProcessor = new BulkProcessor(clientUtil, 100, 5000L) {
        @Override
        public void complete() {
            System.out.println("All documents processed successfully");
        }

        @Override
        public void error(Exception exception) {
            System.err.println("Error during batch processing: " + exception.getMessage());
        }
    };

    try {
        for (int i = 0; i < 1000; i++) {
            Document doc = new Document();
            doc.setId(String.valueOf(i));
            doc.setTitle("Document " + i);
            doc.setContent("Content for document " + i);

            bulkProcessor.insertDocument("documents", doc);
        }

        bulkProcessor.shutdown();

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

---

## 与其他方案对比

| 对比项         | Bboss                         | Easy-ES                   | RestClient       |
| -------------- | ----------------------------- | ------------------------- | ---------------- |
| 多引擎原生支持 | ✓（ES/Easysearch/Opensearch） | Easysearch 原生分支       | ✗                |
| 学习成本       | 低（DSL 即所学）              | 低（Lambda API）          | 中（需学习 API） |
| 开箱即用       | ✓（Spring Boot 自动配置）     | ✓（Spring Boot 自旋配置） | ✗（需手动配置）  |
| ORM 框架       | ✗（提供 O/R Mapping）         | ✓（完整 ORM）             | ✗                |
| 异步批处理     | ✓（BulkProcessor）            | ✓（框架内置）             | ✓（需自己实现）  |
| 多数据源       | ✓（原生支持）                 | ✗                         | ✓（需自己实现）  |
| 代码量         | 中等                          | 极少                      | 较多             |
| 社区活跃度     | 中等                          | 高（Dromara 社区）        | 官方支持         |

---

## 常见操作示例

### REST API 直接操作

```java
/**
 * 执行原始 HTTP 请求
 */
public void executeRawRequest() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    // 获取集群状态
    String response = clientUtil.executeHttp("_cluster/state?pretty",
                                            ClientInterface.HTTP_GET);
    System.out.println("Cluster state: " + response);

    // 获取索引信息
    String indexInfo = clientUtil.executeHttp("documents",
                                              ClientInterface.HTTP_GET);
    System.out.println("Index info: " + indexInfo);
}

/**
 * 检查索引是否存在
 */
public void checkIndexExists() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    boolean exists = clientUtil.existIndice("documents");
    System.out.println("Index exists: " + exists);
}

/**
 * 删除索引
 */
public void dropIndex() {
    ClientInterface clientUtil = bbossESStarter.getRestClient();

    String response = clientUtil.dropIndice("documents");
    System.out.println("Drop index response: " + response);
}
```

---

## 注意事项

| 注意项         | 说明                                                             |
| -------------- | ---------------------------------------------------------------- |
| **Java 版本**  | 建议 JDK 11+，兼容 JDK 8                                         |
| **HTTPS 配置** | Easysearch 默认启用 HTTPS，配置 `useHttps: true`                 |
| **用户认证**   | 启用认证时需配置 `elasticUser` 和 `elasticPassword`              |
| **索引名称**   | 建议使用小写字母和数字，避免特殊字符                             |
| **文档 ID**    | 需自定义或让 Easysearch 自动生成，建议自定义便于管理             |
| **刷新策略**   | 生产环境建议不使用 `refresh=true`，避免频繁刷新影响性能          |
| **DSL 配置**   | DSL 文件放在 `resources/esmapper` 目录下，Spring Boot 会自动加载 |
| **多数据源**   | 通过 `searchListWithCluster()` 等带 Cluster 后缀的方法指定数据源 |

---

## 调试技巧

```java
/**
 * 打印执行的 JSON 请求
 */
public void enableDebugMode() {
    // 通过配置启用 DSL 打印
    ClientInterface clientUtil = bbossESStarter.getConfigRestClient("esmapper/document.xml");

    // 也可在代码中直接打印返回的 JSON
    String jsonResponse = clientUtil.executeRequest("documents/_search",
                                                    "searchByKeyword",
                                                    params);
    System.out.println("Request JSON: " + jsonResponse);
}
```

---

## 相关链接

- [Bboss 官方网站](https://www.bbossgroups.com/)
- [Bboss 官方文档](https://esdoc.bbossgroups.com/)
- [Bboss Gitee 仓库](https://gitee.com/bboss/bboss-elastic)
- [Bboss Spring Boot 示例](https://gitee.com/bboss/springboot-elasticsearch)
- [Bboss Hello World 示例](https://gitee.com/bboss/eshelloword-booter)

---

## 延伸阅读

- [Java 客户端]({{< relref "/docs/integrations/clients/java.md" >}})
- [Quick Start 教程]({{< relref "/docs/quick-start/tutorial.md" >}})
- [常见问题]({{< relref "/docs/faq/_index.md" >}})


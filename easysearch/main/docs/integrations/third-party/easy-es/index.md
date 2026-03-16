---
title: "Easy-ES 查询框架"
date: 0001-01-01
description: "Easy-ES 与 Easysearch 原生集成，MyBatis-Plus 风格的搜索引擎 ORM 框架。"
summary: "Easy-ES 查询框架 #   Easy-ES 是 Dromara 开源社区下的搜索引擎 ORM 框架，类似于 MyBatis-Plus 之于 MySQL。它在 Java 客户端的基础上只做增强不做改变，提供 Lambda 风格的简洁 API，可减少 50%~80% 的代码量。
Easy-ES 提供了专门的 Easysearch 原生分支，底层直接使用 Easysearch Java Client，无需 ES API 兼容模式，具备完整的原生性能和功能支持。
 源码仓库： https://gitee.com/dromara/easy-es/tree/easy-es4easySearch Maven 中央仓库： easy-es-boot-starter 2.1.0-easysearch  核心优势 #     特性 说明     极简开发 一行代码完成查询，相比原生 API 代码量减少 50%~80%   自动索引管理 索引全生命周期由框架自动托管，零停机更新，无需手动管理   SQL 语法兼容 支持 MySQL 风格的 and、or、like、in 等常用语法查询   Lambda 表达式 类型安全的字段访问，避免手写字段名导致的错误   Spring Boot 集成 开箱即用的自动配置，完美融入 Spring Boot 生态   原生 Easysearch 支持 底层使用 Easysearch Java Client，无需兼容层     快速开始 #  1."
---


# Easy-ES 查询框架

[Easy-ES](https://easy-es.cn/) 是 Dromara 开源社区下的搜索引擎 ORM 框架，类似于 MyBatis-Plus 之于 MySQL。它在 Java 客户端的基础上只做增强不做改变，提供 Lambda 风格的简洁 API，可减少 50%~80% 的代码量。

Easy-ES 提供了专门的 **Easysearch 原生分支**，底层直接使用 Easysearch Java Client，无需 ES API 兼容模式，具备完整的原生性能和功能支持。

- 源码仓库：[https://gitee.com/dromara/easy-es/tree/easy-es4easySearch](https://gitee.com/dromara/easy-es/tree/easy-es4easySearch/)
- Maven 中央仓库：[easy-es-boot-starter 2.1.0-easysearch](https://mvnrepository.com/artifact/org.dromara.easy-es/easy-es-boot-starter/2.1.0-easysearch)

## 核心优势

| 特性 | 说明 |
|------|------|
| **极简开发** | 一行代码完成查询，相比原生 API 代码量减少 50%~80% |
| **自动索引管理** | 索引全生命周期由框架自动托管，零停机更新，无需手动管理 |
| **SQL 语法兼容** | 支持 MySQL 风格的 `and`、`or`、`like`、`in` 等常用语法查询 |
| **Lambda 表达式** | 类型安全的字段访问，避免手写字段名导致的错误 |
| **Spring Boot 集成** | 开箱即用的自动配置，完美融入 Spring Boot 生态 |
| **原生 Easysearch 支持** | 底层使用 Easysearch Java Client，无需兼容层 |

---

## 快速开始

### 1. 添加依赖

#### Maven

```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <spring-boot.version>2.7.0</spring-boot.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.dromara.easy-es</groupId>
        <artifactId>easy-es-boot-starter</artifactId>
        <version>2.1.0-easysearch</version>
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

sourceCompatibility = '11'

dependencies {
    implementation 'org.dromara.easy-es:easy-es-boot-starter:2.1.0-easysearch'
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

### 2. 配置连接

```yaml
# application.yml
easy-es:
  enable: true
  address: localhost:9200
  schema: https
  username: admin
  password: your_password_here
  keep-alive-millis: 18000

  global-config:
    process-index-mode: smoothly
    async-process-index-blocking: true
    print-dsl: false
    db-config:
      map-underscore-to-camel-case: true
      id-type: customize
      field-strategy: not_empty
      refresh-policy: immediate
      enable-track-total-hits: true
```

**配置说明**：

| 参数 | 说明 |
|------|------|
| `address` | Easysearch 服务地址（`host:port`） |
| `schema` | 协议，Easysearch 默认启用 HTTPS，填 `https` |
| `username` / `password` | Easysearch 用户认证凭据 |
| `keep-alive-millis` | 连接保持时间（毫秒） |
| `process-index-mode` | 索引处理模式：`smoothly`（平滑零停机）、`manual`（手动） |
| `print-dsl` | 是否打印 DSL 语句（开发调试时设为 `true`） |
| `refresh-policy` | 刷新策略：`immediate`（立即）、`wait_until`、`none` |

### 3. 定义实体类

```java
import org.dromara.easyes.annotation.*;
import org.dromara.easyes.annotation.rely.*;

@Settings(shardsNum = 3, replicasNum = 2)
@IndexName(value = "easyes_document", keepGlobalPrefix = true)
public class Document {

    @IndexId(type = IdType.CUSTOMIZE)
    private String id;

    /** 标题 - 默认 keyword 类型 */
    private String title;

    /** 内容 - TEXT 类型，使用 IK 分词器，支持高亮 */
    @HighLight(mappingField = "highlightContent")
    @IndexField(fieldType = FieldType.TEXT, analyzer = Analyzer.IK_SMART)
    private String content;

    /** 创建者 */
    @IndexField(strategy = FieldStrategy.NOT_EMPTY)
    private String creator;

    /** 创建时间 */
    @IndexField(fieldType = FieldType.DATE, dateFormat = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime gmtCreate;

    /** 高亮结果映射字段 */
    private String highlightContent;

    /** 点赞数 */
    private Integer starNum;

    /** 地理位置 */
    @IndexField(fieldType = FieldType.GEO_POINT)
    private String location;

    // getters & setters ...
}
```

**常用注解**：

| 注解 | 说明 |
|------|------|
| `@IndexName` | 指定索引名，`keepGlobalPrefix` 启用全局前缀 |
| `@IndexId` | 主键标识，`IdType.CUSTOMIZE` 为自定义 ID |
| `@IndexField` | 字段映射，指定字段类型、分词器、更新策略等 |
| `@HighLight` | 高亮配置，`mappingField` 指定高亮结果映射的字段 |
| `@Settings` | 索引设置，分片数、副本数等 |

### 4. 定义 Mapper

```java
import org.dromara.easyes.core.kernel.BaseEsMapper;

public interface DocumentMapper extends BaseEsMapper<Document> {
}
```

### 5. 启动类配置

```java
import org.dromara.easyes.spring.annotation.EsMapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EsMapperScan("com.example.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 6. 业务使用

```java
import org.dromara.easyes.core.conditions.select.LambdaEsQueryWrapper;
import org.easysearch.action.search.SearchResponse;

@RestController
public class DocumentController {

    @Resource
    private DocumentMapper documentMapper;

    /** 插入文档 */
    @GetMapping("/insert")
    public Integer insert() {
        Document doc = new Document();
        doc.setId("1");
        doc.setTitle("Easy-ES 入门");
        doc.setContent("这是一篇关于 Easy-ES 与 Easysearch 集成的教程");
        doc.setCreator("admin");
        doc.setGmtCreate(LocalDateTime.now());
        doc.setStarNum(100);
        return documentMapper.insert(doc);
    }

    /** Lambda 条件查询 */
    @GetMapping("/search")
    public List<Document> search(@RequestParam String keyword) {
        LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
        wrapper.match(Document::getContent, keyword)
               .ge(Document::getStarNum, 10)
               .orderByDesc(Document::getStarNum);
        return documentMapper.selectList(wrapper);
    }

    /** 高亮搜索 */
    @GetMapping("/highlight")
    public List<Document> highlight(@RequestParam String content) {
        LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
        wrapper.match(Document::getContent, content);
        return documentMapper.selectList(wrapper);
    }

    /** 聚合查询 */
    @GetMapping("/agg")
    public Aggregations aggregation() {
        LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
        wrapper.groupBy(Document::getGmtCreate)
               .max(Document::getStarNum)
               .min(Document::getStarNum);
        SearchResponse response = documentMapper.search(wrapper);
        return response.getAggregations();
    }

    /** SQL 查询 */
    @GetMapping("/sql")
    public String queryBySQL(@RequestParam String title) {
        String sql = String.format(
            "SELECT * FROM dev_easyes_document WHERE title = '%s'", title);
        return documentMapper.executeSQL(sql);
    }
}
```

---

## 与 ES 通用版的区别

| 对比项 | Easysearch 原生版 | ES 通用版 |
|--------|-------------------|-----------|
| Maven 坐标 | `org.dromara.easy-es:easy-es-boot-starter:2.1.0-easysearch` | `org.dromara.easy-es:easy-es-boot-starter:2.x.x` |
| 底层客户端 | Easysearch Java Client | Elasticsearch RestHighLevelClient |
| API 兼容模式 | **不需要** | 需要开启 `elasticsearch.api_compatibility: true` |
| 源码分支 | `easy-es4easySearch` | `master` |
| import 包名 | `org.easysearch.*` | `org.elasticsearch.*` |

> 如果您从 Easy-ES 的 ES 通用版本迁移到 Easysearch 原生版，只需更换 Maven 依赖并将 import 中的 `org.elasticsearch` 替换为 `org.easysearch`。

---

## 注意事项

| 注意项 | 说明 |
|--------|------|
| Java 版本 | 需要 JDK 11 及以上 |
| Spring Boot 版本 | 推荐 2.7.x |
| HTTPS | Easysearch 默认启用 HTTPS，配置 `schema: https` |
| 认证 | 配置 `username` 和 `password` |
| 索引前缀 | 通过 `db-config.index-prefix` 设置全局索引前缀 |

## 相关链接

- [Easy-ES 官方网站](https://easy-es.cn/)
- [Easy-ES Easysearch 分支源码](https://gitee.com/dromara/easy-es/tree/easy-es4easySearch/)
- [Easy-ES 2.1.0-easysearch 发布公告](https://infinilabs.cn/blog/2025/easy-es-2.1.0-easysearch-release/)
- [Maven 中央仓库](https://mvnrepository.com/artifact/org.dromara.easy-es/easy-es-boot-starter/2.1.0-easysearch)

## 延伸阅读

- [Java 客户端]({{< relref "/docs/integrations/clients/java.md" >}})
- [Quick Start 教程]({{< relref "/docs/quick-start/tutorial.md" >}})


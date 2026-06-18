---
title: "SQL-JDBC"
date: 0001-01-01
summary: "SQL-JDBC 驱动 #  Easysearch SQL JDBC 驱动程序是一个独立的纯 Java 驱动，可将 JDBC 调用转换为 Easysearch SQL REST API 请求，适用于 BI 工具集成、报表生成和应用程序数据访问等场景。
 驱动信息 #     属性 值     Driver 类名 com.easysearch.jdbc.EasysearchDriver   JDBC URL 前缀 jdbc:easysearch://   JAR 包 sql-jdbc-1.7.1.jar   最低 Java 版本 Java 8     安装 #  下载 #  从官网下载 JDBC 驱动 JAR：
https://release.infinilabs.com/easysearch/archive/plugins/sql-jdbc-1.7.1.jar Maven 项目 #  将 JAR 安装到本地仓库后引用："
---


# SQL-JDBC 驱动

Easysearch SQL JDBC 驱动程序是一个独立的纯 Java 驱动，可将 JDBC 调用转换为 Easysearch SQL REST API 请求，适用于 BI 工具集成、报表生成和应用程序数据访问等场景。

---

## 驱动信息

| 属性 | 值 |
|------|-----|
| Driver 类名 | `com.easysearch.jdbc.EasysearchDriver` |
| JDBC URL 前缀 | `jdbc:easysearch://` |
| JAR 包 | `sql-jdbc-1.7.1.jar` |
| 最低 Java 版本 | Java 8 |

---

## 安装

### 下载

从官网下载 JDBC 驱动 JAR：

```
https://release.infinilabs.com/easysearch/archive/plugins/sql-jdbc-1.7.1.jar
```

### Maven 项目

将 JAR 安装到本地仓库后引用：

```xml
<dependency>
    <groupId>com.easysearch</groupId>
    <artifactId>sql-jdbc</artifactId>
    <version>1.7.1</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/libs/sql-jdbc-1.7.1.jar</systemPath>
</dependency>
```

### Gradle 项目

将 JAR 放入 `libs/` 目录，在 `build.gradle` 中添加：

```groovy
plugins {
    id 'java'
}

repositories {
    mavenCentral()
    flatDir { dirs 'libs' }
}

dependencies {
    implementation files('libs/sql-jdbc-1.7.1.jar')
}
```

---

## 连接 URL 格式

```
jdbc:easysearch://[scheme://]host[:port][/path][?param=value&...]
```

| 组成 | 说明 | 默认值 |
|------|------|-------|
| `scheme` | 协议（`http` 或 `https`） | `http` |
| `host` | Easysearch 节点地址 | `localhost` |
| `port` | 端口号 | `9200` |
| `path` | 可选的请求路径前缀 | — |

**示例：**

```
jdbc:easysearch://localhost:9200
jdbc:easysearch://https://es-node.example.com:9210
jdbc:easysearch://https://localhost:9210?ssl=true&trustSelfSigned=true
```

---

## 连接属性

通过 `Properties` 对象或 URL 查询参数传递：

### 基本属性

| 属性 | 说明 | 默认值 |
|------|------|-------|
| `user` | 用户名 | — |
| `password` | 密码 | — |
| `fetchSize` | 每次获取的行数（启用游标分页） | `0`（不启用） |

### 认证属性

| 属性 | 说明 | 默认值 |
|------|------|-------|
| `auth` | 认证类型 | `BASIC`（提供 user/password 时） |

支持的 `auth` 值：

| 值 | 说明 |
|-----|------|
| `NONE` | 无认证 |
| `BASIC` | HTTP Basic 认证（需提供 `user`/`password`） |
| `AWS_SIGV4` | AWS Signature V4 认证 |

### SSL/TLS 属性

| 属性 | 说明 | 默认值 |
|------|------|-------|
| `ssl` | 是否启用 SSL | `false` |
| `trustSelfSigned` | 是否信任自签名证书 | `false` |
| `hostnameVerification` | 是否验证主机名 | `true` |
| `trustStoreLocation` | 信任库文件路径（JKS/PKCS12） | — |
| `trustStorePassword` | 信任库密码 | — |
| `keyStoreLocation` | 客户端密钥库路径（双向 TLS） | — |
| `keyStorePassword` | 客户端密钥库密码 | — |

---

## 使用示例

### 基本连接与查询

```java
import java.sql.*;
import java.util.Properties;

public class EasysearchJdbcExample {
    public static void main(String[] args) throws Exception {
        String url = "jdbc:easysearch://https://localhost:9210";

        Properties props = new Properties();
        props.put("user", "admin");
        props.put("password", "admin");
        props.put("ssl", "true");
        props.put("trustStoreLocation", "/path/to/truststore.jks");
        props.put("trustStorePassword", "changeit");
        props.put("hostnameVerification", "false");

        try (Connection conn = DriverManager.getConnection(url, props);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(
                 "SELECT firstname, lastname, age FROM accounts WHERE age > 30 ORDER BY age")) {

            while (rs.next()) {
                System.out.printf("Name: %s %s, Age: %d%n",
                    rs.getString("firstname"),
                    rs.getString("lastname"),
                    rs.getInt("age"));
            }
        }
    }
}
```

### 使用 PreparedStatement

```java
String sql = "SELECT * FROM accounts WHERE age > ? AND gender = ?";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setInt(1, 30);
    pstmt.setString(2, "M");
    try (ResultSet rs = pstmt.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("firstname"));
        }
    }
}
```

### 游标分页（大结果集）

通过 `fetchSize` 启用游标分页，避免一次加载全部结果：

```java
Properties props = new Properties();
props.put("user", "admin");
props.put("password", "admin");
props.put("fetchSize", "500");  // 每批获取 500 行

try (Connection conn = DriverManager.getConnection(url, props);
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery("SELECT * FROM large_index")) {
    while (rs.next()) {
        // 自动分批获取
    }
}
```

### 获取结果集元数据

```java
ResultSetMetaData meta = rs.getMetaData();
int columnCount = meta.getColumnCount();
for (int i = 1; i <= columnCount; i++) {
    System.out.printf("Column %d: %s (%s)%n",
        i, meta.getColumnName(i), meta.getColumnTypeName(i));
}
```

---

## 自签名证书配置

在开发环境中使用自签名证书时，有两种方式：

### 方式一：信任自签名证书（简便）

```java
props.put("ssl", "true");
props.put("trustSelfSigned", "true");
props.put("hostnameVerification", "false");
```

### 方式二：使用信任库（推荐用于生产环境）

先将证书导入 JKS 信任库：

```bash
keytool -importcert -alias easysearch \
  -file /path/to/server-cert.pem \
  -keystore /path/to/truststore.jks \
  -storepass changeit -noprompt
```

然后配置：

```java
props.put("ssl", "true");
props.put("trustStoreLocation", "/path/to/truststore.jks");
props.put("trustStorePassword", "changeit");
```

---

## 注意事项

- JDBC 驱动通过 HTTP/HTTPS 与 Easysearch 的 `/_sql` REST API 通信
- 默认使用 `jdbc` 输出格式，返回结构化的 schema + datarows
- 设置 `fetchSize > 0` 会启用 Scroll API 进行游标分页
- 驱动不完全符合 JDBC 规范，部分高级 JDBC 功能可能不可用

---

## 相关链接

- [SQL 查询总览]({{< relref "/docs/features/sql/" >}})
- [SQL 客户端集成]({{< relref "/docs/integrations/clients/sql.md" >}})


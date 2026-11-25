---
title: "插件开发入门"
date: 0001-01-01
description: "基于官方模板快速完成第一个插件的完整闭环。"
summary: "插件开发入门 #  本页带你从克隆模板到本地验证，跑通第一个 Easysearch 插件。
1. 环境准备 #  Easysearch 2.5.2 的插件构建和测试要求 JDK 21 或更高版本。使用模板附带的 Gradle wrapper，最低版本为 8.10.2。
# 检查 Java java -version # 检查 Gradle ./gradlew -v 2. 从官方模板初始化项目 #  git clone https://github.com/infinilabs/easysearch-plugin-template.git my-plugin cd my-plugin 重命名包名和类名后即可开始开发。项目结构说明见：结构与类型。
3. 配置 build.gradle（插件元信息） #  plugin-descriptor.properties 由构建过程自动生成。以下以 Easysearch 2.5.2 为例，请选择与目标 Easysearch 一致的 build-tools 版本，并确认 Maven 仓库提供同版本制品。使用自建制品库时，将其地址同时加入 buildscript.repositories 和项目的 repositories。
buildscript { ext { easysearchVersion = System.getProperty(&#34;easysearch.version&#34;, &#34;2.5.2&#34;) } repositories { mavenCentral() gradlePluginPortal() maven { url &#34;https://maven."
---


# 插件开发入门

本页带你从克隆模板到本地验证，跑通第一个 Easysearch 插件。

## 1. 环境准备

Easysearch 2.5.2 的插件构建和测试要求 JDK 21 或更高版本。使用模板附带的 Gradle wrapper，最低版本为 8.10.2。

```bash
# 检查 Java
java -version

# 检查 Gradle
./gradlew -v
```

## 2. 从官方模板初始化项目

```bash
git clone https://github.com/infinilabs/easysearch-plugin-template.git my-plugin
cd my-plugin
```

重命名包名和类名后即可开始开发。项目结构说明见：[结构与类型]({{< relref "./architecture-and-types.md" >}})。

## 3. 配置 `build.gradle`（插件元信息）

`plugin-descriptor.properties` 由构建过程自动生成。以下以 Easysearch 2.5.2 为例，请选择与目标 Easysearch 一致的 build-tools 版本，并确认 Maven 仓库提供同版本制品。使用自建制品库时，将其地址同时加入 `buildscript.repositories` 和项目的 `repositories`。

```groovy
buildscript {
    ext {
        easysearchVersion = System.getProperty("easysearch.version", "2.5.2")
    }
    repositories {
        mavenCentral()
        gradlePluginPortal()
        maven { url "https://maven.aliyun.com/repository/central" }
        maven { url "https://maven.aliyun.com/repository/gradle-plugin" }
    }
    dependencies {
        classpath "com.infinilabs.easysearch.gradle:build-tools:${easysearchVersion}"
    }
}

apply plugin: 'easysearch.esplugin'

repositories {
    mavenCentral()
}

esplugin {
    name 'my-plugin'
    description 'My Easysearch Plugin'
    classname 'com.example.myplugin.MyPlugin'
    licenseFile rootProject.file('LICENSE.txt')
    noticeFile rootProject.file('NOTICE.txt')
}

group = 'com.example'
version = '0.1.0'

// Skip POM validation - this plugin is not published to maven.
tasks.named('validateNebulaPom') { enabled = false }
```

使用带自动接线的 build-tools 时，`easysearch.esplugin` 会为 Gradle 测试加载同版本安全 agent。模板中的旧手工接线可在升级时移除。具体迁移步骤见 [测试与验证]({{< relref "./testing-and-validation.md" >}})。

## 4. 编写插件入口类

最简单的插件只需继承 `Plugin`：

```java
package com.example.myplugin;

import org.easysearch.plugins.Plugin;

public class MyPlugin extends Plugin {

    public MyPlugin() {
        // 可在此做初始化日志
    }
}
```

需要 `AnalysisPlugin`、`ActionPlugin`、`IngestPlugin` 等扩展类型时，参考：[结构与类型]({{< relref "./architecture-and-types.md" >}})。

## 5. 构建

```bash
./gradlew clean build

# 产物路径
ls build/distributions/
# my-plugin-0.1.0.zip
```

## 6. 安装并验证

```bash
# 安装插件（需要绝对路径）
bin/easysearch-plugin install \
  file:///$(pwd)/build/distributions/my-plugin-0.1.0.zip

# 启动 Easysearch
bin/easysearch

# 确认插件已加载
curl -s "http://localhost:9200/_cat/plugins?v" | grep my-plugin
```

## 7. 快速迭代循环

```bash
# 改代码后
./gradlew assemble                                 # 跳过测试快速构建
bin/easysearch-plugin remove --purge my-plugin     # 卸载旧版
bin/easysearch-plugin install file:///$(pwd)/build/distributions/my-plugin-0.1.0.zip
# 重启 Easysearch，验证 _cat/plugins
```

## 下一步

- [结构与类型]({{< relref "./architecture-and-types.md" >}})：理解插件接口与类型选型
- [测试与验证]({{< relref "./testing-and-validation.md" >}})：编写测试与调试技巧
- [插件发布与分享]({{< relref "./publishing.md" >}})：打包与发布


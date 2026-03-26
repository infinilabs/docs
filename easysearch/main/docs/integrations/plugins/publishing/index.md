---
title: "插件发布与分享"
date: 0001-01-01
description: "插件打包、分发与安装的实践指南。"
summary: "插件发布与分享 #  打包 #  ./gradlew clean build # 查看产物 ls -lh build/distributions/ # 输出示例：my-plugin-0.1.0.zip # 确认 zip 内容 unzip -l build/distributions/my-plugin-0.1.0.zip # 应包含： # my-plugin-0.1.0.jar # plugin-descriptor.properties 安装到 Easysearch #  # 从本地路径安装 bin/easysearch-plugin install file:///ABSOLUTE/PATH/build/distributions/my-plugin-0.1.0.zip # 从 URL 安装（发布到 GitHub Releases 后） bin/easysearch-plugin install https://github.com/yourname/my-plugin/releases/download/v0.1.0/my-plugin-0.1.0.zip 重启 Easysearch 后验证：
bin/easysearch curl -s http://localhost:9200/_nodes/plugins | \  python3 -m json.tool | grep -A3 &#34;my-plugin&#34; # 预期输出： # &#34;name&#34;: &#34;my-plugin&#34;, # &#34;version&#34;: &#34;0."
---


# 插件发布与分享

## 打包

```bash
./gradlew clean build

# 查看产物
ls -lh build/distributions/
# 输出示例：my-plugin-0.1.0.zip

# 确认 zip 内容
unzip -l build/distributions/my-plugin-0.1.0.zip
# 应包含：
#   my-plugin-0.1.0.jar
#   plugin-descriptor.properties
```

## 安装到 Easysearch

```bash
# 从本地路径安装
bin/easysearch-plugin install file:///ABSOLUTE/PATH/build/distributions/my-plugin-0.1.0.zip

# 从 URL 安装（发布到 GitHub Releases 后）
bin/easysearch-plugin install https://github.com/yourname/my-plugin/releases/download/v0.1.0/my-plugin-0.1.0.zip
```

重启 Easysearch 后验证：

```bash
bin/easysearch

curl -s http://localhost:9200/_nodes/plugins | \
  python3 -m json.tool | grep -A3 "my-plugin"
# 预期输出：
# "name": "my-plugin",
# "version": "0.1.0",
```

## 卸载插件

```bash
bin/easysearch-plugin remove my-plugin
```

## 发布到 GitHub Releases

```bash
# 1. 打 tag
git tag -a v0.1.0 -m "release v0.1.0"
git push origin v0.1.0

# 2. 用 GitHub CLI 创建 Release 并上传 zip
gh release create v0.1.0 \
  build/distributions/my-plugin-0.1.0.zip \
  --title "v0.1.0" \
  --notes "初始版本"
```

发布后用户可直接通过 URL 安装：

```bash
bin/easysearch-plugin install \
  https://github.com/yourname/my-plugin/releases/download/v0.1.0/my-plugin-0.1.0.zip
```

## 版本管理

遵循语义化版本 `MAJOR.MINOR.PATCH`：

- `PATCH`：bug 修复，向后兼容
- `MINOR`：新增功能，向后兼容
- `MAJOR`：有破坏性变更

建议在每次发布时维护 `CHANGELOG.md`：

```markdown
## [0.2.0] - 2026-03-24
### 新增
- 支持 xxx 字段类型

## [0.1.0] - 2026-03-01
### 新增
- 初始版本
```

## 发布前检查清单

- [ ] `plugin-descriptor.properties` 字段完整，`easysearch.version` 与目标版本一致
- [ ] `./gradlew test` 和 `./gradlew integTest` 均通过
- [ ] 本地安装验证：`_nodes/plugins` 能看到插件
- [ ] README 写明安装命令、兼容版本与已知限制
- [ ] CHANGELOG.md 已更新

## 相关文档

- [开发与测试]({{< relref "./dev-and-test.md" >}})
- [插件开发入门]({{< relref "./getting-started.md" >}})


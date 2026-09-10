---
title: "新增规则引擎"
date: 0001-01-01
description: "在 Easysearch UI 中新增规则引擎。"
summary: "新增规则引擎 #  通过新增规则引擎，可以创建规则库并编译规则，使其可被 Ingest Pipeline 引用。
操作步骤 #    进入规则引擎页面：在左侧导航栏点击「规则引擎」，进入规则库列表页面。
  打开新建规则库弹窗：点击页面右上角的「+ 新建」按钮，弹出「新建规则库」配置窗口。
   配置规则库信息：
 填写必填项「规则库 ID」，自定义规则库唯一标识（如示例中的 test_rule_engine_id）。 填写必填项「名称」，用于展示规则库名称（如示例中的 test_rule_engine_name）。 （可选）填写「描述」，说明规则库用途。 在「规则列表」中填写 JSON 数组格式的规则配置，每条规则需包含 expression 和 description。 （可选）点击「+ 标签」添加标签，便于后续分类和筛选。    创建规则库：确认配置无误后，点击「创建」按钮。
  编译规则库：保存成功后，系统会提示规则已保存。点击「编译」按钮，进入规则库编译窗口。  配置编译信息：在「编译规则库」窗口中，按需配置「字段声明」和「联合索引」等信息，然后点击「编译」按钮。  查看编译结果：编译成功后，系统会展示 Ingest Pipeline 创建示例，可点击「复制命令」复制引用当前规则库的管道配置。  查看创建结果：返回规则引擎列表后，可查看新增规则库信息。示例中规则库 test_rule_engine_id 状态为「已编译」，规则总数为 1，版本为 1。  注意事项 #   规则库 ID 不可与已有规则库重复，否则创建将失败。 规则列表需使用合法 JSON 数组格式，且每条规则需包含 expression 和 description 字段。 规则保存后需要完成编译，发布后的规则才可被 Ingest Pipeline 加载并识别。 编译完成后，请根据页面提供的示例创建或更新 Ingest Pipeline，使写入流程引用当前规则库。  "
---


# 新增规则引擎

通过新增规则引擎，可以创建规则库并编译规则，使其可被 Ingest Pipeline 引用。

## 操作步骤

1. **进入规则引擎页面**：在左侧导航栏点击「规则引擎」，进入规则库列表页面。

2. **打开新建规则库弹窗**：点击页面右上角的「+ 新建」按钮，弹出「新建规则库」配置窗口。

{{% load-img "/img/management/rule-engine/create-rule-engine/image-1.png" %}}

3. **配置规则库信息**：
   - 填写必填项「规则库 ID」，自定义规则库唯一标识（如示例中的 `test_rule_engine_id`）。
   - 填写必填项「名称」，用于展示规则库名称（如示例中的 `test_rule_engine_name`）。
   - （可选）填写「描述」，说明规则库用途。
   - 在「规则列表」中填写 JSON 数组格式的规则配置，每条规则需包含 `expression` 和 `description`。
   - （可选）点击「+ 标签」添加标签，便于后续分类和筛选。

4. **创建规则库**：确认配置无误后，点击「创建」按钮。

{{% load-img "/img/management/rule-engine/create-rule-engine/image-2.png" %}}

5. **编译规则库**：保存成功后，系统会提示规则已保存。点击「编译」按钮，进入规则库编译窗口。

{{% load-img "/img/management/rule-engine/create-rule-engine/image-3.png" %}}

6. **配置编译信息**：在「编译规则库」窗口中，按需配置「字段声明」和「联合索引」等信息，然后点击「编译」按钮。

{{% load-img "/img/management/rule-engine/create-rule-engine/image-4.png" %}}

7. **查看编译结果**：编译成功后，系统会展示 Ingest Pipeline 创建示例，可点击「复制命令」复制引用当前规则库的管道配置。

{{% load-img "/img/management/rule-engine/create-rule-engine/image-5.png" %}}

8. **查看创建结果**：返回规则引擎列表后，可查看新增规则库信息。示例中规则库 `test_rule_engine_id` 状态为「已编译」，规则总数为 `1`，版本为 `1`。

{{% load-img "/img/management/rule-engine/create-rule-engine/image-6.png" %}}

## 注意事项

- 规则库 ID 不可与已有规则库重复，否则创建将失败。
- 规则列表需使用合法 JSON 数组格式，且每条规则需包含 `expression` 和 `description` 字段。
- 规则保存后需要完成编译，发布后的规则才可被 Ingest Pipeline 加载并识别。
- 编译完成后，请根据页面提供的示例创建或更新 Ingest Pipeline，使写入流程引用当前规则库。


---
title: "集群设置"
date: 0001-01-01
description: "动态集群设置的查看和管理。"
summary: "集群设置 #  「集群设置」页面用于查看和修改 Easysearch 的动态集群设置，即通过 _cluster/settings API 管理的配置项。索引模板、节点 easysearch.yml 等静态配置不在此页面范围内。
登录 Easysearch UI，在左侧导航栏点击「设置」，默认进入「集群设置」标签页。
页面按「节点」「索引」「分片」「断路器」「管道管理」「主从复制」「生命周期」「备份管理」「集群」「日志与元数据」等分组展示全部设置项，右侧为分组导航卡片，可点击快速跳转。
搜索与过滤 #   搜索框：支持按设置名称或 key 实时过滤（如输入 recovery 或 indices.recovery.max_bytes_per_sec）；点击「刷新」按钮可重新拉取集群的最新设置。   来源过滤：右侧下拉框可按「全部 / 临时 / 持久 / 默认」过滤，只看某一层的设置项。  查看设置的当前有效值和所有的值 #  在 Easysearch 中，一个设置项的值分为三层：
   层 徽标颜色 说明     临时（transient） 金色 集群重启后失效   持久（persistent） 蓝色 持久化到 cluster state，重启后仍生效   默认（default） 灰色 内置默认值，只读    三层取值的优先级为：临时 &gt; 持久 &gt; 默认。列表每一行展示设置名称、类型标签（bool、int、time、size 等）、完整 key 和当前生效值，生效值右侧的徽标标明其来源层。"
---


# 集群设置

「集群设置」页面用于查看和修改 Easysearch 的**动态集群设置**，即通过 `_cluster/settings` API 管理的配置项。索引模板、节点 easysearch.yml 等静态配置不在此页面范围内。

登录 Easysearch UI，在左侧导航栏点击「设置」，默认进入「集群设置」标签页。

{{% load-img "/img/management/settings/cluster-settings/image-1.png" %}}

页面按「节点」「索引」「分片」「断路器」「管道管理」「主从复制」「生命周期」「备份管理」「集群」「日志与元数据」等分组展示全部设置项，右侧为分组导航卡片，可点击快速跳转。

## 搜索与过滤

- **搜索框**：支持按设置**名称或 key** 实时过滤（如输入 `recovery` 或 `indices.recovery.max_bytes_per_sec`）；点击「刷新」按钮可重新拉取集群的最新设置。

  {{% load-img "/img/management/settings/cluster-settings/image-2.png" %}}
- **来源过滤**：右侧下拉框可按「全部 / 临时 / 持久 / 默认」过滤，只看某一层的设置项。

  {{% load-img "/img/management/settings/cluster-settings/image-3.png" %}}

## 查看设置的当前有效值和所有的值

在 Easysearch 中，一个设置项的值分为三层：

| 层 | 徽标颜色 | 说明 |
|---|---|---|
| 临时（transient） | 金色 | 集群重启后失效 |
| 持久（persistent） | 蓝色 | 持久化到 cluster state，重启后仍生效 |
| 默认（default） | 灰色 | 内置默认值，只读 |

三层取值的优先级为：**临时 > 持久 > 默认**。列表每一行展示设置名称、类型标签（bool、int、time、size 等）、完整 key 和**当前生效值**，生效值右侧的徽标标明其来源层。

点击任意一行可展开查看该设置项的全部取值：临时、持久、默认三张并排的卡片，每张卡片标注了该层的状态——「生效中」「被覆盖」或「未设置」。默认层为只读；当临时或持久层正在生效时，默认值暂不可读取，卡片显示为「—」，清除覆盖后即可见。

{{% load-img "/img/management/settings/cluster-settings/image-4.png" %}}

## 更改集群设置

1. 找到要更改的设置项（可借助搜索框），点击该行展开。
2. 在「临时」或「持久」卡片内输入新值。控件随设置类型自动适配：bool/enum 为下拉选择，int/float 为数字输入，list 为标签输入（逗号分隔），time/size 等需保留单位的类型为文本输入（如 `60s`、`40mb`）。未设置的卡片会以默认值作为占位提示。
3. 点击「保存修改」。修改将影响集群的全部节点。

同一设置项可以在一次保存中同时修改临时层与持久层；有输入未保存时切换页面或刷新将丢失草稿。

## 新增动态设置项

部分设置的名称本身是动态的，无法枚举展示，需要先「添加」实例后才能设置值，包括：日志级别（`logger.*`）、分片分配属性（include/exclude/require）、断路器（`breaker.*`）、脚本上下文（`script.context.*`）等。

在对应分组标题右侧点击「添加」按钮，输入实例名称（如日志器名 `org.easysearch.http` 或分配属性名 `zone`）并填写取值即可。

{{% load-img "/img/management/settings/cluster-settings/image-5.png" %}}

## 清空设置的值

临时/持久的值都可以清空，即将该层置为 NULL：

- **清除单层**：展开设置项后，点击对应卡片底部的「清除此层」链接。清除后生效值将回落到更低优先级层（持久值或默认值）。
  {{% load-img "/img/management/settings/cluster-settings/image-6.png" %}}
- **清空临时**：点击页面右上角的「清空临时」按钮，一键移除「临时」层的**全部**设置。此操作会弹出确认框，确认后生效值将集体回落到持久值或默认值，不可撤销。
  {{% load-img "/img/management/settings/cluster-settings/image-7.png" %}}


## 所需权限

查看本页需要集群监控权限，修改与清空需要 `cluster:admin/settings/update` 权限；无权限时页面仅只读展示。


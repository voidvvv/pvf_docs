---
title: 来源摘要：my_inclusion.md（nut/PVF 入门）
category: source
tags: [nut, 入门, passiveobject, appendage]
sources: ["input/my_inclusion.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 来源摘要：my_inclusion.md（nut / PVF 入门）

> 原文：`input/my_inclusion.md`（源工作区内）。nut / PVF 结构 / appendage / 被动对象的入门文档。

## 核心结论（3~5 句）

1. nut 是 DNF 客户端 / 服务端共析的 Squirrel 脚本，靠钩子函数扩展游戏；目录按职业与职能组织，每技能一目录（`{技能名}.nut` + `po_*` + `ap_*`）；
2. `{职业}_load_state.nut` 是注册入口（pushScriptFiles / pushState / pushPassiveObj），header 文件集中定义 STATE_* / SKILL_* / CUSTOM_ANI_* 常量；
3. appendage 两种定义（.apd 声明式可被装备 `[appendage]` 标签引用 / nut 钩子式），change status 与 dummy 是最常用类型；
4. 被动物 = `.obj` + `po_*.nut` + `passiveobject.lst` 三件套，技能通过 `sq_SendCreatePassiveObjectPacket` 创建、`setCustomData` 按序读自定义数据。

## 已生成的 wiki 页

| 页面 | 提取内容 |
|------|----------|
| [[nut-scripting]] | 目录结构、文件类型表、技能钩子流程 |
| [[load-state]] | 注册原语与调用方式 |
| [[appendage]] | .apd 格式、钩子全表、API |
| [[passive-object]] | 三件套、数据传递、.obj 格式 |
| [[pvf-format]] | 根目录布局（部分） |

## 原文中未搬运、需回原文的细节

- 各钩子的完整示例函数体（`proc_appendage_atmage_effect` 等）；
- header 常量的完整定义列表（元素枚举、SKL_* 索引常量等）；
- appendage 钩子表的少数低频条目（`getImmuneTypeDamageRate`、`onSourceKeyFrameFlag` 等）。

---
title: 来源摘要：快捷栏装备系统与 Appendage 使用指南
category: source
tags: [快捷栏, 纹章, appendage, 排错]
sources: ["快捷栏装备系统与Appendage使用指南.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 来源摘要：快捷栏装备系统与 Appendage 使用指南

> 原文：`快捷栏装备系统与Appendage使用指南.md`（源工作区根目录）。基于 `huanjing/sqr` 实际源码逐文件分析；排错结论已实测确认。

## 核心结论（3~5 句）

1. 快捷栏装备（勇者纹章 100000101~110）完全由 sqr 实现：`drawMainCustomUI` / `isUsableItem` / `procAppend_<job>` 三函数靠 6 槽共享向量 `itemIndex` 协作，每帧同步挂 / 卸 ap；
2. Appendage 有声明式（.apd + lst）与脚本式（ap_*.nut 钩子）两种定义，`CNSquirrelAppendage` API 家族统一管理，改属性走 `sq_AddChangeStatus` + `addParameter`；
3. **procAppend_Priest 纹章失效的根因是前缀 `useRangeBuff` 每帧抛 null 解引用、被引擎静默捕获而中断函数**——"无报错但不工作"在 sqr 中的标准解释；
4. 本服叠加多个模组命名空间（js60_qq506807329 / apjh / qq358332886_monster_appendage / unclebang / custom）。

## 已生成的 wiki 页

| 页面 | 提取内容 |
|------|----------|
| [[quickbar-equipment-system]] | 三函数机制、文件清单、关键代码位 |
| [[crest-items]] | 纹章道具族对照 |
| [[add-quickbar-equipment]] | 新增四步流程与模板 |
| [[appendage]] | 两种定义、API 全表、钩子、陷阱 |
| [[procappend-priest-bug]] | 完整排错案例与方法论 |
| [[load-state]] | 加载顺序实测（附录 A.3） |
| [[nut-scripting]] | 模组命名空间（附录 A.1） |

## 原文中未搬运、需回原文的细节

- 三函数的完整源码（`isUsableItem` 的 O(n²) 去重写法、`drawMainCustomUI` 全文）；
- 方案 B 修复的完整 `useRangeBuff_getTeamCount` 代码；
- `ap_100000110.nut` 全文（多属性模板完整版）；
- 附录 A.2 函数索引表的其余行。

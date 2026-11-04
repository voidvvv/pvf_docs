---
title: 快捷栏装备系统（三函数协作机制）
category: entity
tags: [快捷栏, 纹章, equipment, appendage]
sources: ["快捷栏装备系统与Appendage使用指南.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 快捷栏装备系统（三函数协作机制）

"快捷栏装备"是放进**快捷栏（消耗品栏）就生效、拿出就失效**的特殊道具（本服表现为勇者纹章，见 [[crest-items]]）。效果**不靠道具文件自带的 `[append]` 段**（`.stk` 里没有），完全由 sqr 脚本每帧扫描快捷栏状态、动态挂 / 卸对应 [[appendage]] 实现。

## 涉及文件

| 文件 | 作用 |
|---|---|
| `stackable/crest/1000001xx.stk` | 纹章道具定义（名称 / 图标 / 说明），**无 `[append]` 段** |
| `equipment/ap/ap_1000001xx.nut` | 对应 ap 脚本（定义加什么属性） |
| `equipment/equipment_<job>.nut` | 每职业一份，含三函数 |
| `equipment/equipment_main.nut` | 加载器，`sq_RunScript` 所有 `equipment_<job>.nut` |

## 三函数协作（核心）

靠角色身上的共享向量 `obj.getVar("itemIndex")`（6 槽）传值：

```
┌──────── obj.getVar("itemIndex")  ← 6 槽共享向量 ────────┐
   ▲ 每帧重置              ▲ 使用时写入             ▲ 每帧读取
   │                       │                        │
drawMainCustomUI_<job>   isUsableItem_<job>      procAppend_<job>
（重置成 [0..5] 空槽标志）  （把纹章码写进空槽）      （扫描→在栏挂 ap/离栏卸 ap）
```

- **`drawMainCustomUI_<job>(obj)`**：每帧把向量重置为 `[0,1,2,3,4,5]`；"空槽"用 `值 == 下标` 识别。
- **`isUsableItem_<job>(obj, itemIndex)`**：引擎在使用道具时调用；把道具码写入第一个空槽（带全表去重）。已知瑕疵：内层 `local i` 遮蔽外层、O(n²) 冗余扫描（不影响功能）。
- **`procAppend_<job>(obj)`**：每帧对每个纹章码执行"在栏 → `sq_AppendAppendage` 挂载 + `sq_Append` 提交；离栏 → `GetSquirrelAppendage(路径).setValid(false)` 摘除"（完整代码模板见 [[add-quickbar-equipment]]）。

## 关键代码位（源工作区实测）

| 函数 | 文件 | 行 |
|---|---|---|
| `procAppend_Priest` | `equipment/equipment_priest.nut` | 38 |
| `isUsableItem_Priest` | `equipment/equipment_priest.nut` | 2 |
| `drawMainCustomUI_Priest` | `equipment/equipment_priest.nut` | 24 |
| `procAppend_AllGrowJob` | `apjh/procappend.nut` | 1 |
| `isUsableItem_AllGrowJob` / `duegonUsableItem` | `apjh/lue.nut` | 39 / 58 |
| `drawNewDamageUI_AllGrowJob` | `apjh/drawmaincustomui.nut` | 1 |

## 已知问题

- **`100000110` 段写坏**（`equipment_priest.nut`）：调试残留导致 `sq_AppendAppendage` 跑到 `if` 外——**不论在不在快捷栏都被挂上且摘不掉**；修法：照 `100000101` 模板改回。详见 [[add-quickbar-equipment]]。
- `procAppend_<job>` 前缀调用抛异常会阻断纹章挂载 → [[procappend-priest-bug]]。

## 相关

- 新增一件快捷栏装备的步骤：[[add-quickbar-equipment]]
- 道具族：[[crest-items]]　·　机制基础：[[appendage]]
- 来源摘要：[[src-quickbar-guide]]

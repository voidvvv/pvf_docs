---
title: 案例：procAppend_Priest 纹章失效（静默异常阻断）
category: debugging
tags: [排错, appendage, 静默异常, priest]
sources: ["快捷栏装备系统与Appendage使用指南.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 案例：procAppend_Priest 纹章失效（静默异常阻断）

> 根因已**实测确认**：`procAppend_Priest` 前缀里的 `useRangeBuff(obj)` 每帧抛运行时异常（null 解引用），在纹章代码**之前**中断了整个函数。"单独去掉 `useRangeBuff` 即恢复"即为证据。

## 现象

圣职者把勇者纹章放进快捷栏后，纹章属性**不生效**，且**没有任何报错**。

## 根因机制

- DNF 引擎在 C++/Squirrel 边界**静默捕获** Squirrel 异常：不崩溃、不提示，但本次函数调用就此中止；
- 纹章的 `sq_AppendAppendage` 位于 `procAppend_Priest` 末尾，前缀一抛，末尾整段本帧不执行 → 永不挂载。

**抛点**：`useRangeBuff` → `useRangeBuff_getTeamCount`（`character/new_priest/rangebuff/rangebuff.nut:31`）：

| 抛点 | 触发条件 |
|---|---|
| `objectManager.getCollisionObjectNumber()` | `obj.getObjectManager()` 返回 null（角色过渡态） |
| `sq_IsAiCharacter(sqrChr)` | 对象转 SQRCharacter 返回 null（NPC、召唤物、销毁中的队友） |

致命细节：`useRangeBuff_getTeamCount` 在 `rangeBuff` 开关判断**之前**无条件调用——从没用过群体 BUFF 也在每帧跑。

## 排查过程（方法论价值）

1. **先验证链路各环节**（ap 文件 ✅ / 道具 ✅ / .stk 无 [append] 兜底 ❌ 符合设计 / 职业间一致 ✅ / 钩子被调用 ✅）——排除"资源缺失"类假设；
2. 弯路：曾假设"`itemIndex` 向量写入时机问题"——**被推翻**。教训：先验证执行流跑不跑得到，再怀疑数据；
3. **二分法锁定元凶**：先删前缀 5 个调用 → 恢复；再逐个加回 → 只有 `useRangeBuff(obj)` 一加就坏。

## 修复方案

- **方案 A（当前采用）**：`procAppend_Priest` 中只去掉 `useRangeBuff(obj)` 一行，保留其余 4 个前缀调用。代价：失去群体 BUFF 功能。
- **方案 B（保留群体 BUFF）**：给 `useRangeBuff_getTeamCount` 加 null 保护（`if(!obj) return array;`、`if(!objectManager) return array;`、`if(!sqrChr) continue;`），再把调用加回去。两全其美。

## 经验教训（通用）

1. **"不工作且无报错"** 在 DNF sqr 里 ≈ 脚本抛了被静默吞掉的异常，优先怀疑执行流被中断；
2. 定位手法：可疑函数加 `print("[DBG] …")` 埋点看每帧停在哪行；或前缀调用逐个加减二分；
3. 写每帧函数（procAppend / drawMainCustomUI）时，**任何解引用、外部对象遍历都要 null 保护**——一次 null 让整个函数本帧失效；
4. **不要霰弹枪式修复**："5 个全删"能修好但误杀功能；隔离出唯一元凶后只动它一个。

## 相关

- 机制背景：[[quickbar-equipment-system]]、[[appendage]]（陷阱 §5）
- 静默失败姊妹篇：[[naming-traps]]
- 来源摘要：[[src-quickbar-guide]]

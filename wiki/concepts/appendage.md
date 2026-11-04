---
title: Appendage（附加状态）机制与 API
category: concept
tags: [appendage, ap, nut, buff]
sources: ["快捷栏装备系统与Appendage使用指南.md", "input/my_inclusion.md", "AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# Appendage（附加状态）机制与 API

Appendage（简称 **ap**）是挂载到游戏对象（角色 / 怪物 / 被动体）上的**状态效果载体**：BUFF、被动、装备特效、纹章属性都靠它。一个 ap 关联一个 `.nut` 脚本，由 `CNSquirrelAppendage` 统一管理。

特点：独立于角色 state（挂上持续生效，与技能状态机无关）；可设持续时间到时自灭，也可手动 `setValid(false)` 摘除；ap 也可以本身无效果（dummy 标记），供别处读取判定。

## 两种定义方式

| 方式 | 做法 | 引用途径 |
|------|------|----------|
| **声明式** `.apd` | `appendage/` 下 `.apd` 文件，在 `appendage.lst` 注册唯一 ID | 装备 `[appendage]` 标签按 ID 引用（穿戴 / 事件触发判定） |
| **脚本式** `.nut` | `sqr/**/ap_*.nut`，`sq_AddFunctionName` 注册钩子 | `sq_AppendAppendage` 按脚本路径挂载 |

### .apd 文件格式（声明式）

```
[type]        `change status`          # change status | attack type | change basic attack type | dummy
[duration]    180000                   # 毫秒，0=永久
[buff]        1                        # 1=增益 0=减益
[max overlap] 1                        # 最大叠加层数
[string data] `physical attack` …      # 属性名列表
[float data]  100.0 …                  # 与 string data 一一对应的修改值
[/…]
```

`change status`：string 指定属性、float 指定值（如 `attack speed` + `10.0` = +10% 攻速）。
`dummy`：不改任何属性，仅作标记供脚本检测。

### ap 脚本结构（脚本式）

```squirrel
function sq_AddFunctionName(appendage){
    appendage.sq_AddFunctionName("onStart", "onStart_ap_XXX");   // 必须注册
    appendage.sq_AddFunctionName("onEnd",   "onEnd_ap_XXX");
    appendage.sq_AddFunctionName("proc",    "proc_ap_XXX");      // 可选钩子
}
function onStart_ap_XXX(appendage){
    if(!appendage) return;
    local obj = appendage.getParent();
    if(!obj){ appendage.setValid(false); return; }   // 宿主没了自删（必须保护）
    local change = appendage.sq_getChangeStatus("XXX");
    if(!change) change = appendage.sq_AddChangeStatus("XXX", obj, obj, 0, 37, false, 500);
    if(change){
        change.clearParameter();
        change.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_ATTACK, false, 500.tofloat());
    }
}
```

`sq_AddChangeStatus(key, src, tgt, 0, 37, isPercent, baseValue)`：第 4/5 位内部类型码照搬 `0, 37`；`addParameter(类型, 是否百分比, 值.tofloat())`。

## 可用钩子（部分）

`proc`（每帧核心逻辑）、`onStart` / `onEnd`（挂上 / 摘除）、`prepareDraw`、`drawAppend`（额外绘制）、`isEnd`（返回 true 则结束）、`onStartMap`、`onVaildTimeEnd`（到期）、`onDamageParent` / `onAttackParent`（宿主受击 / 攻击）、`onSetHp` / `onApplyHpDamage`、`onChangeState`、`onDestroyObject` 等。

## 核心 API

| API | 作用 |
|-----|------|
| `CNSquirrelAppendage.sq_AppendAppendage(tgt, src, skillId, isChangeState, apPath, isAppendStart)` | 创建 ap；`isAppendStart=true` 创建即激活 |
| `CNSquirrelAppendage.sq_Append(ap, parent, source)` | 提交激活（配合 false 创建的延迟模式） |
| `CNSquirrelAppendage.sq_IsAppendAppendage(obj, apPath)` | 查询是否已挂（**挂前必查，防叠挂**） |
| `CNSquirrelAppendage.sq_RemoveAppendage(obj, apPath)` | 摘除 |
| `appendage.getParent()` / `getSource()` | 宿主 / 来源对象 |
| `appendage.setValid(false)` | 使失效（摘除） |
| `appendage.sq_SetValidTime(ms)` | 持续时间 |
| `appendage.setAppendCauseSkill(BUFF_CAUSE_SKILL, job, skillId, level)` | 登记来源技能 |
| `appendage.getVar()` / `getVar(name)` | ap 自带存储向量（onStart/onEnd 间传值） |

`apPath` 相对 `sqr/` 根，如 `"equipment/ap/ap_100000101.nut"`。

## 两种挂载模式

- **即时**：`sq_AppendAppendage(obj, obj, -1, false, "xxx/ap.nut", true)` —— 简单场景；
- **延迟**（创建→配置→提交）：先 `false` 创建，再 `sq_SetValidTime` / `setAppendCauseSkill`，最后 `sq_Append` 提交——适合觉醒 / 爆发态（实例：`character/common/burster/burster.nut:280-292`）。

## 常用 CHANGE_STATUS_TYPE_*

`PHYSICAL/MAGICAL_ATTACK`、`PHYSICAL/MAGICAL_DEFENSE`、`EQUIPMENT_PHYSICAL/MAGICAL_ATTACK`、`ADDITIONAL_PHYSICAL/MAGICAL_GENUINE_ATTACK`（独立攻击）、`ELEMENT_ATTACK_ALL`（全属强）、`PHYSICAL/MAGICAL_CRITICAL_DAMAGE_RATE`、`STUCK`（命中，负值减命中）。

## ⚠️ 陷阱

1. `isAppendStart` 别搞反：延迟模式忘 `sq_Append` 会挂不上；
2. `onStart` 必须保护 `getParent()` 为 null 的情况；
3. 不调 `sq_SetValidTime` 的 ap 可能瞬时失效——持久效果要么不设时长、要么外部 `setValid` 控制（纹章"在栏才挂"模式）；
4. ap 路径写错会**静默**挂不上；
5. **挂 ap 的宿主函数（procAppend 等）若在挂载代码之前抛异常，挂载代码永远跑不到**——见 [[procappend-priest-bug]]。

## 相关

- 应用实例：[[quickbar-equipment-system]]、[[crest-items]]　·　新增流程：[[add-quickbar-equipment]]
- 来源摘要：[[src-quickbar-guide]]、[[src-my-inclusion]]

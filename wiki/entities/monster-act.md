---
title: 实体：.act 动作文件（TRIGGER/BEHAVIOR 行为语言）
category: entity
tags: [act, trigger, behavior, 怪物, 行为]
sources: ["pvfDoc/pvfDoc/【12】高级教程/ACT/12051_act简单解析.json",
         "pvfDoc/pvfDoc/【13】obj/被动对象/11612_Obj关于ACT的部分进阶文档 — 条件对象.json",
         "pvfDoc/pvfDoc/【12】高级教程/ACT/12053_装备与怪物联动.json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 实体：.act 动作文件（TRIGGER/BEHAVIOR 行为语言）

`.act` 是声明式怪物行为程序（25,352 个，50.2% 纯动画无触发器）。顶层仅三种块，交替排列、无名，**靠全文件 0 基序号互相引用**：

```
[MOTION]  动画/音效/台词层
[TRIGGER] 条件 → [DO BEHAVIOR] 目标 n   （n=全文件第 n 个 BEHAVIOR）
[BEHAVIOR] 行为块（指令字典见 [[monster-behavior-commands]]）
```

## [MOTION]

- `[BASE ANI]` 主动画；`[SUB ANI]` `路径 起始帧 图层`（图层负=背景层）；
- `[SOUND]` `名称 播放帧`；`[SPEECH]` `台词 帧 权重`（气泡台词，须繁体）。

## [TRIGGER] 条件语法

骨架：`[TRIGGER] 条件… [DO BEHAVIOR] 目标 索引 [/TRIGGER]`

| 条件类 | 写法 | 说明 |
|---|---|---|
| 帧区间 | `[FRAME] a b` / `[FRAME JUST] n` | 动画第 a~b 帧触发（最常用） |
| 事件 | `[ON SET ACTION]` / `[ON ATTACKSUCCESS]` / `[ON DAMAGE]` / `[ON END ACTION]` / `ON COUNTER HIT` / `ON DESTROY OBJECT` / `ON MOVE COLLISION` / `ON CHANGE LIFE` 等 | 动作开始/命中/被击/结束… |
| 限次 | `[LIMIT] n`（0=无限） | 阶段防重入 |
| 周期 | `[CHECK TIME] n`（ms） / `[PROC]` | 每 n ms 轮询 / 常驻逐帧 |
| 门控 | `[ENABLE] [ON]/[OFF]` | 触发器初始启停态 |
| 对象筛选 | `[WHICH] 选择器 …` | 见下 |
| 变量 | `[IF] 变量 [+] 7500 [<=] [GET DUNGEON TIME]` | 配合 DIM |

**[WHICH] 子语法**（=doc 11612 全表）：
`[WHICH] <ME/TARGET/CHARACTER/MONSTER/PASSIVE/ALL ENEMY/ALL CHARACTER TEAM/LAST ATTACKSUCCESS/…> ([GET TARGET] n [RANDOM|DISTANCE HIGH/LOW]) ([CHECKUP] 逐对象条件 [/CHECKUP])`
CHECKUP 内：`[IS INDEX] id…`（怪物/obj ID）、`DISTANCE`/`ZPOS`/`HP <op> n [%]`（血量门槛）、`IS TARGET STATE`、`NOT HAVE FOLLOWING OBJECT [PASSIVE] id` 等。
检出对象由 `[DO BEHAVIOR] [CHECKUP OBJECT] n` 逐个执行行为（上下文锚定到该对象位置——瞬移贴脸/在目标脚下放物全靠它）。

**[DO BEHAVIOR] 目标**：`[ME] n`（37,378 次，本文件第 n 个 BEHAVIOR）为主；另 TARGET/CHECKUP OBJECT/LAST ATTACKSUCCESS 等对类目标直接作用。一个 TRIGGER 可挂多个 DO BEHAVIOR；`[RANDOM SELECT] 0 1 2`=分支随机；`[DELAY DO BEHAVIOR][ME] n ms`=延迟连段。

## 变量与阶段机

- `[DIM]` `变量` `[=] [GET TIME]`（1,794 次）定义计时变量，`[+]/[-]` 算术；`GET TIME`=动作计时、`GET DUNGEON TIME`=地下城总时长；
- **BEHAVIOR 内嵌 `[TRIGGER] n [ON]/[OFF]`（7,767 次）= 开关本文件第 n 个触发器**——连锁/阶段机的核心写法（召唤后关 0 开 1、变身时整批关闭一阶段逻辑）；
- `[DUNGEON SYMBOL SET/COMPARE]`=跨对象门控变量；`[SET ACTION CUSTOM]` 给 .obj/召唤物切状态。

## 坑

1. **`[ME] n` 是全文件 0 基序号**，不是"紧随本 TRIGGER 的那个"——新手第一坑；102 个原版文件存在越界引用（引擎静默忽略，勿模仿）；
2. 闭合式块（`[SET SPEED]…[/SET SPEED]`）与兄弟标签式（`[SET ACTION][CUSTOM]6[NOW]`）并存，解析须按缩进隐式闭合；
3. 引擎对未知标签宽容（typo `RSSET`/`MaY`/`CHECKTIME` 静默忽略）——拼错=行为悄悄不生效；
4. 比较符 `>=`/`=>`、`<=`/`=<` 冗余并存。

## 实战样本

`huanjing/monster/110levelnormaldungeon/vericollis/boss/michael/`（16 招机制 Boss）、`newmonsters/powerstation/fuhrman/`（解析成文：源工作区 `demo/fuhrman_机制考核报告.md`）。

## 相关

[[monster-behavior-commands]]（指令字典）· [[monster-dir]] · [[monster-ai]] · [[modify-monster]]

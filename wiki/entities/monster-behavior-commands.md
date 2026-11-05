---
title: 实体：BEHAVIOR 行为指令字典
category: entity
tags: [behavior, 指令, act, 怪物, 速查]
sources: ["pvfDoc/pvfDoc/【12】高级教程/ACT/12051_act简单解析.json",
         "pvfDoc/pvfDoc/【12】高级教程/ACT/12052…12058 模板系列.json",
         "pvfDoc/pvfDoc/【12】高级教程/ACT/12085…json",
         "pvfDoc/pvfDoc/【03】怪物修改/10336_怪物变身原理.json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 实体：BEHAVIOR 行为指令字典

`.act` 内 [BEHAVIOR] 块的指令词汇表（2026-11-05 全量普查 25,352 个 .act、约 180 种指令的浓缩版；证据：=doc=pvfDoc 明确 / 无标=用例自证 / ？=推测）。**改机制=查这张表拼指令**。

## 位移类

| 指令 | 参数 | 含义 |
|---|---|---|
| `TELEPORT` | `[POS] x y z` + `MOVE ME`/`MOVE TARGET`/`RANDOM`/`ALLOW FIXTURE` | 瞬移（=doc 12056） |
| `SET SPEED` | `[X/Z/Y AXIS] n` + `USE MY DIRECTION` | 冲刺/跳跃（速度驱动位移） |
| `INTERPOLATION MOVE TARGET` | OFFSET/MOVE TYPE/FRAME/ACCEL/PARABOLA… | 向目标插值移动（可抛物线） |
| `SET POS FORCE` | X/Y/Z START…END + MOVE ME/UNIFORM | 强制坐标区间位移 |
| `MOVE TO TARGET` / `MOVE PATTERN` / `JUMP TO TARGET [EX]` | — | 走向目标 / 轨道移动 / 跳向目标（EX：JUMP HIGH/MOVE DISTANCE…） |

## 判定类

| 指令 | 含义 |
|---|---|
| `ATTACKRECT [RESET]/[ON]/[OFF]` | 命中框开关/重置；**框体与伤害在 .atk**，act 只管时机 |
| `CHANGE ATTACKINFO` `路径.atk` | .act 内换 .atk 的唯一指令 |
| `SET DAMAGE BOX [ON]/[OFF]` | 受击框开关，**OFF=无敌**（=doc 12057） |
| `COUNTER HIT INFO` `[ID]0 [FRAME]a~b` | 指定可被反击帧窗口 |

## 召唤/衍生类

| 指令 | 含义 |
|---|---|
| `SUMMON MONSTER` `[INDEX] lst号 [LEVEL] [POS]` + MAGIC CIRCLE/SUMMON TIME/SUMMON MARK… | 召怪（ID=monster.lst） |
| `SUMMON APC` | 召 APC（INDEX=aicharacter 号） |
| `CREATE PASSIVEOBJECT` `[INDEX] passiveobject.lst号 [PARTICLE FILENAME] .ptl [POS]` + `USE MAP POS`(按地图坐标铺场)/`FOLLOWING TARGET`/`RANDOM`/`WARNING MARK`(预警圈)/`FIX DIRECTION` | 万物皆可放：弹道/机关/地面物/预警（=doc 12051） |
| `CREATE PASSIVEOBJECT CIRCLE` | 环形放置 |
| `CHANGE DISGUISER` `怪物ID 时长` | 变身（=doc 10336，可把玩家变怪） |
| `DESTROY` | 销毁目标（对 ME=自毁） |

## 状态类

| 指令 | 含义 |
|---|---|
| `APPENDAGE` `时长ms` + 属性名 % 组 | 限时属性状态（如 物理/魔法 damage reduce -50 =受伤+50%） |
| `RESTORE [HP]/[MP]` `n`/`[%]` | 回血；**负数=扣血（无视无敌的真伤）**（=doc 12055） |
| `SET HP` `n` | 设绝对 HP（0=自杀退场） |
| `VIRTUALHP APPENDAGE` | 锁血（999 999999999 2 10） |
| `ACTIVE STATUS` `[stun]/[burn]/[poison]/[freeze]/[stone]/[slow]/[hold]…` + `几率 等级 时间` | 上异常（=doc 12085） |
| `REMOVE ACTIVE STATUS` / `CHARACTER COOLTIME RESET` | 清异常 / 重置目标冷却 |
| `ADD/REMOVE CATEGORY` `fixture`/`non attraction`… | 碰撞类别（fixture=定身物） |
| `SET ABSOLUTE SUPER ARMOR` `1` | 绝对霸体 |
| `CAMOUFLAGE` `1 时长ms` | 隐身 |
| `CHANGE AI` `NORMAL/ALL` | 恢复/切换 AI |
| `SET ACTION` `宾语 n [NOW]` | 切动作：ATTACK n→[attack action] / CUSTOM n→[etc action] / STAND/DOWN/DIE…（0 基，=doc 12053） |
| `CHANGE ACTION FILE` `[STAY/MOVE/ATTACK/DOWN/DAMAGE 0|1]` `file.act` | 整类换动作文件（受击轻重分流/变身全套换） |
| `SET FRAME` `n` | 跳帧（循环/压前摇） |
| `SET DIRECTION` `TO TARGET/REVERSE/RANDOM…` | 朝向 |
| `ADD/REMOVE LAYER`、`COLOR CHANGE EFFECT`、`OBJECT SCALE EX`、`SET VISIBLITY`、`NAME HIDE ON/OFF` | 表现层开关 |

## 演出/系统类

| 指令 | 含义 |
|---|---|
| `CASTING` `时长ms 类型`（EX 加强版可换读条 UI）/`CANCEL CASTING`/`TARGET CASTING CANCEL` | 读条（类型 1 常见；ABSOLUTE=不可打断?） |
| `FLASH SCREEN`、`SHAKING 强度 ms`、`PLAY SOUND`、`SAY SPEECH`（WITH POSITION）、`DIALOG 台词 ms`、`NOTICE`、`CENTER MSG` | 反馈演出 |
| `CUT SCENE`、`ZOOM IT (PLUS/OUT)`、`OZMA CAMERA` | 过场/镜头 |
| `DUNGEON SYMBOL SET`、`SET SYMBOL`、`CALC` | 地牢级门控变量 |
| `START/END KEYSTROKE MODE` `8 9 1 2 3` | QTE |
| `SEND MESSAGE` `[ETC_COMMAND]/[DIE_COMMAND]` | 事件命令（DIE=令退场） |
| `PULL APPENDAGE (EX)` `x z ms`（负=推开，=doc 12058）、`WIND/RESOLUTION/TIME CHANGE APPENDAGE`（250 10 10=时缓） | 特殊位移/时间 |
| `MUCU LIMIT CONTROL` `10000` | 禁魔 10 秒（=doc 12054） |
| `DIM 变量 = GET TIME`、`TRIGGER n ON/OFF` | 变量/触发器开关（阶段机） |

## 用例速查（写机制时照抄）

- 无敌窗口：`[SET DAMAGE BOX][OFF]` … `[ON]`；高级无敌 `IMPROVED INVINCIBLE TIME 10000 HIT EFFECT OFF`；
- 血线触发：TRIGGER 内 `[WHICH][ME][CHECKUP][HP][<=]20[%]`；或 .mob `[event super on scr]`；
- 铺场：`CREATE PASSIVEOBJECT [INDEX] xxx [USE MAP POS]` 逐坐标摆放（michael 光柱）；
- 贴脸连段：WHICH CHARACTER GET TARGET 1 RANDOM → DO BEHAVIOR CHECKUP OBJECT（TELEPORT/放物）；
- 双段判定：同帧 `DO BEHAVIOR ME n` + `DELAY DO BEHAVIOR ME n 50ms`。

## 相关

[[monster-act]]（语法骨架）· [[monster-dir]] · [[modify-monster]]（配方总入口）

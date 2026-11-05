---
title: 流程：修改怪物机制（需求→配方速查）
category: howto
tags: [howto, 怪物, 修改, 机制, 配方]
sources: []
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 流程：修改怪物机制（需求→配方速查）

> **用法**：用户说出"想要什么机制/改什么"→ 在下表找到配方 → 按"落点"去改。定位怪：monster.lst 反查（两行 ID 在前）或 search_pvf 搜名字。改完 `import_file` + `save_as_pvf`（见 [[pvfutility-mcp]]）。
> 语法骨架见 [[monster-act]]，指令全表见 [[monster-behavior-commands]]，字段表见 [[monster-dir]]/[[monster-atk]]/[[monster-ai]]。

## 数值类

| 需求 | 落点 |
|---|---|
| 加/减血量、攻击、防御 | `.mob [ability category]` 倍率（`[HP MAX] * 1300`）；按等级整体变 → 基础表 tbl（[[monster-dir]] 三层模型） |
| 某副本怪变强/变弱 | `.dgn [monsterapc diff table]` 换难度表，或 monsterapcdifficultybonus.tbl 对应难度列 |
| 技能伤害 | 先判型：.atk `[absolute damage]` 是占位 → 改 `[damage bonus]`% 或 mob 倍率；直伤型 → 改 absolute（[[monster-atk]]） |
| 等级 | .map `[monster]` 行第 2-3 位等级规则（`1 0` 吃 dgn / `0 N` 锁死）或 dgn `[basis level]` |
| 出手频率 | `[cooltime]` 对应动作的值调小 / `[attack kind]` 列 1 概率调大 / `[think term]` 调小 |
| 掉落 | `.mob [item]`（`物品ID 掉率`，基数推定 1,000,000）/ champion/death tower 系段 / 掉率三件套见 [[modify-drop-rate]] |

## 行为类

| 需求 | 落点 |
|---|---|
| 加一个新技能 | `.mob [attack action]` 追加行 + 对齐 `[attack info]`/`[cooltime]` + 写 .act（MOTION+TRIGGER/BEHAVIOR）+ .atk + Action.ai 加分支（模板抄 [[add-monster]] §2） |
| 换/删技能 | Action.ai 删分支 + [attack action] 行（注意三表同步缩行） |
| 血线换技能表 | Action.ai 入口 `[hp] %>= n` + `[import ai]` 分流（fuhrman 三段式实证） |
| 读条技能 | .act 内 `CASTING 时长ms 类型`；不可打断读条用 ABSOLUTE（未证） |
| 无敌窗口 | `SET DAMAGE BOX OFF…ON`；高级无敌 `IMPROVED INVINCIBLE` |
| 霸体/免打断 | .mob `[category]` 加对应标签；帧级 `SET ABSOLUTE SUPER ARMOR 1` |
| 血线/定期霸体 | .mob `[event super on scr]`（如 80/60/40/20% 各 5 秒） |
| 召唤小怪 | .act `SUMMON MONSTER [INDEX] lst号 [POS]`（小怪=独立 .mob，须已注册） |
| 铺地板机关/预警圈 | `CREATE PASSIVEOBJECT [INDEX] obj号 [USE MAP POS]/[WARNING MARK]`（obj 查 passiveobject.lst，走 MCP） |
| 瞬移贴脸 | WHICH CHARACTER GET TARGET 1 RANDOM → DO BEHAVIOR CHECKUP OBJECT（TELEPORT/攻击） |
| 阶段机/机制轮转 | BEHAVIOR 内 `TRIGGER n ON/OFF` 开关链 + `[LIMIT]`/`[DIM]`+`[IF]` 计时（[[monster-act]] §变量与阶段机） |
| 扣玩家真伤% | .act `RESTORE [HP] -20 [%]`（对 CHECKUP 检出玩家执行） |
| 控制/上异常 | .atk `[active status]`；act 侧 `ACTIVE STATUS [stun] 概率 等级 时间` |
| 变身 | `CHANGE DISGUISER 怪物ID 时长` + `CHANGE ACTION FILE` 换整套动作（=doc 10336） |
| 锁血 | `VIRTUALHP APPENDAGE` |
| 台词/演出 | `[SPEECH]`（气泡，须繁体）/`SAY SPEECH`/`DIALOG`/`NOTICE`/`FLASH SCREEN`/`SHAKING` |
| 攻速/移速 | `[attack speed]/[move speed]`（1000=100% 动画速率）；战斗中变速 `SET SPEED` |

## 打感/受击类

| 需求 | 落点 |
|---|---|
| 能否被浮空 | `[weight]`（单位 10g）/`[floating height]` |
| 硬直长短 | `[hit recovery]`（0=2 秒）；轻重受击分流 `CHANGE ACTION FILE [DAMAGE] 0/1` |
| 被打反应 | .atk `[damage reaction]` + `[push aside]/[lift up]/[knuck back]` |
| 抗控程度 | 各 `[* resistance]`（200≈免控、负=特意可控） |

## 机制只对某难度生效

AI 四件套按难度分文件（heroaction.ai 替换）；或 `[level of difficulty]` per 难度补正。

## 红线复查（每次改完过一遍）

1. 三表行数仍对齐；`[ME] n`/`TRIGGER n` 序号未越界；
2. 新引用的 ID 都已注册（monster/passiveobject lst）；
3. 原版 typo/大小写照抄（[[naming-traps]]）；
4. save_as_pvf + 换文件（[[pvfutility-mcp]] 的 Script(1).pvf 坑）；
5. 游戏内实测每招（引擎对拼错标签静默忽略——没报错≠生效）。

## 相关

[[add-monster]] · [[modify-dungeon-map]] · [[dungeon-chain]]

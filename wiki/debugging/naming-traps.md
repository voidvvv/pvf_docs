---
title: 陷阱集：原版拼写与命名红线
category: debugging
tags: [陷阱, 命名, 排错]
sources: ["AGENTS.md", "物品掉率文件修改说明.md"]
created: 2026-11-04
updated: 2026-11-06
status: current
---

# 陷阱集：原版拼写与命名红线

**原版文件名 / 字段名中的"错误"拼写是游戏识别的一部分。任何"顺手纠正"都会导致游戏无法识别或表错位，且通常无报错。**

## 已知案例

| 原版写法 | 错误"纠正" | 后果 |
|----------|-----------|------|
| `itemdropinfo_monseter.etc` | `itemdropinfo_monster.etc` | 游戏找不到掉率表（`monseter` 是原版 typo，三个掉率文件中两个沿用） |
| `itemdropinfo_monseter_extra.etc` | `…_monster_extra…` | 同上 |
| 难度码 `1000001` / `1000002` | 当成数值改写 | 是副本 / 难度引用码，改动导致掉落表错位 |
| worlddrop 头部第 2 列 `0` | 当成掉率修改 | 保留位，恒为 0 |
| `kingsruinsficultybonus.tbl` | 补成 difficulty | 难度表文件名 typo，.dgn [monsterapc diff table] 按名引用 |
| .act 指令 typo `RSSET` / `USE MaY DIRECTION` / `CHECKTIME` | 顺手修正 | 引擎对未知标签静默忽略——原版 typo 也"不生效"，但改了可能与其他引用不一致 |
| nut API `sq_GetDuegonIndex()` | Duegon→Dungeon | API 原文就是 Duegon，纠正=函数不存在 |
| 目录名 `DarkelfCemetary` / `VilligeOfTemptation` | 改正拼写 | 原版 typo 目录名，改了断链 |

## 大小写与编码陷阱（2026-11-05/06 怪物体系探索新增）

1. **大小写双标**：lst 写 `Act1/Lorien.dgn` 而磁盘是 `act1/lorien.dgn`；`.mob` 引用 `Action/proc1.act` 而目录是 `action/`——Windows 靠大小写不敏感兜底，**新写引用时大小写尽量与磁盘一致**，避免换工具/环境断链；
2. **编码四重奏**：`monster.kor.str`=Big5（名不副实的繁中）、`mob_module_separation_help.txt`=CP949、`character/*/ai/movecommand.lua`=UTF-16 LE、map 的 `.str`=CP949/CP932——直接读必乱码，用 iconv/指定编码；
3. **`.ai` 文件被 Read 工具误判二进制**（实为 CRLF 文本）——用 bash cat 读；
4. **同 section 重复后值覆盖**（.mob 实证）——同名段写两遍不是错误，引擎取后者；
5. 语义陷阱：`[sit action]`=**起身**不是坐下；`[ready action]` 不是动作槽而是"etc motion 索引+叠特效"结构块。

## 防范规则

1. 引用 / 复制原版文件名时**逐字符保持原样**，即使在文档里显得"拼错了"；
2. 新增文件时先查同目录既有命名惯例（如 appendage 用 `ap_` 前缀、被动对象用 `po_` 前缀，见 [[nut-scripting]]）；
3. 新增对象（appendage / 被动对象 / 道具 / 技能）必须同步在对应 `.lst` 注册 ID（见 [[pvf-format]]）；
4. 拿不准字段含义 → 查 `pvfDoc/`（[[pvfdoc-library]]）或对照实际文件，**不臆造**。

## 相关

- 修改红线语境：[[etc-table-format]]、[[modify-drop-rate]]
- 静默失败类问题（无报错但不生效）：[[procappend-priest-bug]]
- 来源摘要：[[src-agents-md]]

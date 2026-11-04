---
title: 陷阱集：原版拼写与命名红线
category: debugging
tags: [陷阱, 命名, 排错]
sources: ["AGENTS.md", "物品掉率文件修改说明.md"]
created: 2026-11-04
updated: 2026-11-04
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

## 防范规则

1. 引用 / 复制原版文件名时**逐字符保持原样**，即使在文档里显得"拼错了"；
2. 新增文件时先查同目录既有命名惯例（如 appendage 用 `ap_` 前缀、被动对象用 `po_` 前缀，见 [[nut-scripting]]）；
3. 新增对象（appendage / 被动对象 / 道具 / 技能）必须同步在对应 `.lst` 注册 ID（见 [[pvf-format]]）；
4. 拿不准字段含义 → 查 `pvfDoc/`（[[pvfdoc-library]]）或对照实际文件，**不臆造**。

## 相关

- 修改红线语境：[[etc-table-format]]、[[modify-drop-rate]]
- 静默失败类问题（无报错但不生效）：[[procappend-priest-bug]]
- 来源摘要：[[src-agents-md]]

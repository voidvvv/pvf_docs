---
title: .equ 装备定义文件格式
category: entity
tags: [equ, 装备, rarity, 格式]
sources: ["pvfDoc/pvfDoc/【02】装备修改/装备属性相关/装备基础组成部分/10347_装备所有标签.json", "pvfDoc/pvfDoc/【02】装备修改/装备属性相关/装备基础组成部分/10343_装备基础属性.json"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# .equ 装备定义文件格式

`.equ` 是装备定义文件，位于 `equipment/` 目录树（角色装备在 `character/<职业>/…`，含 avatar 时装与常规装备），通过 [[equipment-lst]] 注册物品代码。UTF-8 文本、`[section]` 分段（结构通则见 [[etc-table-format]]）。

## 核心字段（含义有证据）

| 字段 | 含义 | 证据 |
|------|------|------|
| `[name]` | 装备名称 | 实测样本 |
| `[basic explain]` | 简述 | 实测样本 |
| `[rarity]` | 稀有度，**单选择型** | pvfDoc《10347_装备所有标签》《10343_装备基础属性》两份一致（映射见下表） |
| `[minimum level]` | 使用最小等级 | 同上（"使用最小等级(单数值型)"） |
| `[maximum level]` | 使用最大等级 | 同上 |
| `[price]` | 向 NPC **买入**价格 | 用户实测告知（2026-11-04），详见 [[npc-price]] |
| `[value]` | 卖给 NPC 价格的**基础值**（实际卖出 = ÷5） | 同上 |
| `[usable job]` … `[/usable job]` | 可用职业（多选，可多行） | pvfDoc《10347》职业枚举表 |
| `[equipment type]` | 部位类型（如 `[ring]` 21，标签+编号两列） | 实测样本 |
| `[item group name]` | 道具分组名 | 实测样本 |

## rarity 稀有度映射（文档证据）

| 值 | 档位 |
|----|------|
| 0 | 白装（普通） |
| 1 | 蓝装（高级） |
| 2 | 紫装（稀有） |
| **3** | **粉（神器）** |
| 4 | 史诗 |
| 5 | 勇者 |

- 引用：`pvfDoc/【02】装备修改/装备属性相关/装备基础组成部分/10347_装备所有标签.json` 与 `10343_装备基础属性.json`（独立来源，数值一致）。
- ⚠️ **文档只记载 0~5**。实测 86幻境存在 `6`（如"深渊之鳞"系列，1,138 件）与 `7`（3 件）、以及 2,036 件无 rarity 字段的 .equ——这些档位**含义未经证实**，勿套用上表猜测。
- ⚠️ 《11595_领主粉的添加教程》警示：.equ 主文件的品级与"装备词典（etc）"里的品级可能被人为设成不一致（词典管词典页分组，主文件管物品颜色）。

## 样本中观察到、含义未证实的字段

`[grade]`（样本值与 minimum level 相同，如 90）、`[attach type]`（`[sealing]`）、`[part set index]`、`[physical/magical attack]`、`[equipment magical defense]`、`[weight]`、`[cool time]`、`[icon]`、`[field image]`、`[move wav]`、`[skill data up]`、`[possible kiri protect]` —— 修改前先查 [[pvfdoc-library]]【02】目录或实测对照，**不要臆造**。

## 实测应用案例

2026-11-04 全量扫描（本地导出副本，非时装 39,707 件）：rarity=3 共 5,740 件，其中 `[minimum level]`≥60 者 3,628 件（结果清单存 `demo/`，gitignore 不入库；等级高峰 86/85/60 级）。方法与坑见 [[pvfutility-mcp]] 实测语义节。

## 相关

- 物品代码索引：[[equipment-lst]]　·　价格规则：[[npc-price]]
- 装备道具族实例：[[crest-items]]、[[quickbar-equipment-system]]
- 来源摘要：[[src-agents-md]]

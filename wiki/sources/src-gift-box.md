---
title: 来源摘要：gift_box_analysis.md（礼盒系统）
category: source
tags: [礼盒, stackable, stk]
sources: ["gift_box_analysis.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 来源摘要：gift_box_analysis.md（礼盒系统）

> 原文：`gift_box_analysis.md`（源工作区根目录）。PVF 礼盒道具（`.stk`，注册于 `stackable/stackable.lst`）的完整分析。

## 核心结论（3~5 句）

1. 礼盒奖励通过**物品代码 ItemCode** 引用，引擎按物品类型在对应 LST（equipment.lst / stackable.lst / creature/pet.lst…）中定位实际文件；
2. `[stackable type]` 决定礼盒行为：`[booster]`（多轮独立随机，每轮抽 1 个、得全部结果，单物品概率 = 自身权重 ÷ 块内权重和）、`[booster random]`（单轮抽 1）、`[booster selection]`（玩家自选）、`[cera package]` / `[usable cera package]`（直得全部）、`[random upgradable legacy]`（多轮可升级）、`[legacy]`（旧版）；
3. 通用字段涵盖名称 / 稀有度 / 图标 / 堆叠 / 冷却 / 绑定类型（`[trade]` 可交易 / `[account]` 绑号）等；限制字段有 `[need material]`（开启材料）、`[expiration date]`（Unix 时间戳）等。

## 待深挖

本文档尚未拆分为独立 wiki 页（当前仅本摘要 + [[pvf-format]] 的 .stk 一行引用）。若开始做礼盒 / 道具类修改，应按 [[WIKI]] §5.1 把"六种礼盒类型详解 + 概率规则"扩为 `entities/gift-box-system` 页。

## 原文中需回原文的细节

- 六种类型的完整 `.stk` 段落示例与概率规则逐条说明；
- `[cera]` 子类型（奖励为点券 / 货币）的写法；
- 通用字段 / 限制字段全表。

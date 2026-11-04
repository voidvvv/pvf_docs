---
title: 来源摘要：atswordman_render_analysis.md（角色渲染链）
category: source
tags: [渲染, chr, ani, atswordman]
sources: ["atswordman_render_analysis.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 来源摘要：atswordman_render_analysis.md（角色渲染链）

> 原文：`atswordman_render_analysis.md`（源工作区根目录）。女鬼剑（ATSwordman）渲染系统分析。

## 核心结论（3~5 句）

1. 角色渲染三层链：`.chr`（角色定义）→ `.ani`（动画帧）→ `.img/.spr`（精灵图）；装备侧 `.equ` → `.lay`（层级脚本）→ `.ani` → 图；
2. `.chr` 定义职业标识（atswordman 为 `[creator mage]`）、转职名列表、身体图模板路径（`%04d` 占位符按帧索引替换，如 `sg_body0009.img`）、待机 / 攻击 / 技能动画列表，以及基础属性、成长、觉醒、技能列表、武器系数；
3. `.ani` 按帧组织：`[FRAME00x]` 内含 `[IMAGE]`（图路径 + 帧索引）、`[IMAGE POS]`（绘制偏移）、`[DELAY]`（帧延迟 ms）、`[DAMAGE BOX]`（受击判定框，可多个）；
4. AT 与普通职业的命名差异成体系：`ATEquipment/` ↔ `Equipment/`、`ATAnimation/` ↔ `Animation/`、`atswordman.lay` ↔ `swordman.lay`。

## 待深挖

尚未拆分独立页。涉及**外观 / 动画 / 新职业渲染**任务时，按 [[WIKI]] §5.1 扩为 `concepts/render-pipeline` 页（.chr/.ani/.lay/.img 详解）。

## 原文中需回原文的细节

- `.lay` 层级脚本的具体语法与装备部位（skin / hair / coat / pants…）目录结构；
- 攻击动画多段 `[DAMAGE BOX]` 参数含义；
- `.chr` 中技能列表与武器技能系数段。

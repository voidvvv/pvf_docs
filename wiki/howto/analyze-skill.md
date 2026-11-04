---
title: 用 dnf-skill-analyzer 分析 .skl 技能
category: howto
tags: [技能, skl, 实操]
sources: ["skills/dnf-skill-analyzer/SKILL.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 用 dnf-skill-analyzer 分析 .skl 技能

技能定义文件 `.skl` 的解析已封装为源工作区的 skill：`skills/dnf-skill-analyzer/SKILL.md`（含 frontmatter，可直接作为 Claude Code skill 加载）。适用于：技能属性 / 伤害 / 冷却 / 等级缩放查询、技能对比、修改前取当前值、按名称（中 / 英）或技能码定位。

## 三步工作流

### 1. 定位技能（按职业选 LST）

| 职业组 | LST 文件 |
|--------|----------|
| 男鬼剑 swordman | `skill/swordmanskill.lst` |
| 女鬼剑 atswordman | `skill/atswordmanskill.lst` |
| 男枪 gunner / 女枪 atgunner | `skill/gunnerskill.lst` / `skill/atgunnerskill.lst` |
| 男格斗 fighter / 女格斗 atfighter | `skill/fighterskill.lst` / `skill/atfighterskill.lst` |
| 男法 mage / 女法 atmage | `skill/mageskill.lst` / `skill/atmageskill.lst` |
| 男圣职 priest / 女圣职 atpriest | `skill/priestskill.lst` / `skill/atpriestskill.lst` |
| 暗夜 thief / 守护 creatormage / 魔枪 demonicswordman | 对应 `skill/*.lst` |

- 有技能码 → `get_lst_file_info` 读 LST（码 → 路径 + 中文名）；
- 有名称 → `search_pvf(keyword="鬼斩", search_folder="skill")`。

### 2. 读取

`get_file_content(file_path="skill/swordman/hardattack.skl")`。

### 3. 解析关键段

| 段 | 含义 |
|----|------|
| `[name]` / `[explain]` / `[basic explain]` | 名称 / 详述 / 简述 |
| `[type]` | `[active]` / `[passive]` |
| `[required level]` / `[maximum level]` / `[purchase cost]` | 学习等级 / 上限 / SP 消耗 |
| `[growtype maximum level]` | 各转职（growtype）下的等级上限 |
| `[level property]` | 等级缩放数据（skill 内置解码流程） |
| `[custom animation]` / 动画索引 | 与 header 的 `CUSTOM_ANI_*` 常量对应（见 [[nut-scripting]]） |

> 深入解码（level property、攻击信息等）以 skill 文档为准，本页只做索引。

## 相关

- 技能脚本能做什么：[[nut-scripting]]　·　技能弹道：[[passive-object]]
- LST 机制：[[pvf-format]]　·　来源摘要：[[src-agents-md]]

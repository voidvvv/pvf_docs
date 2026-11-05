---
title: 实体：怪物 AI（[ai pattern] 四件套与 .ai 语法）
category: entity
tags: [ai, 怪物, 决策树, attack kind]
sources: ["pvfDoc/pvfDoc/【03】怪物修改/10338_怪物全面解读.json",
         "pvfDoc/pvfDoc/【03】怪物修改/11795_AI代码翻译.json",
         "pvfDoc/pvfDoc/【04】技能修改/11735…json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 实体：怪物 AI（[ai pattern] 四件套与 .ai 语法）

`.mob [ai pattern]` 按 5 档难度（easy/medium/hard/ultimate/hero，旧式 normal~slayer）各引 4 个 .ai，**执行序 = .mob 内排列序**：

| 文件 | 职责 | 典型内容 |
|---|---|---|
| Event.ai | 事件反应 | 被击/受异常/`[cube item number]>=1`（无色检测）时反制 |
| DestinationSelect.ai | 走位选择 | return 走位码（0/1/3/10/11…，如 HP 低→11 拉开） |
| MoveMethod.ai | 移动方式 | 绝大多数恒 0（行走） |
| **Action.ai** | **技能选择（核心）** | 条件树 → `return n`（n=[attack action] 索引；-1=待机） |

## .ai 语法

```
[ai pattern]
	[think]
		[void] `条件函数()` 参数…
		[true] … / [false] …        ← 可多层嵌套
		[return] n                   ← n=动作索引 / -1
	[/think]
[/ai pattern]
```

条件函数词表（全库 8,779 个 .ai 普查）：

- `is the skill in cooltime()` n（8,894 次，查 n 号动作 CD）
- `is target in attack area()` x y z w（5,523，距离框；轴义未定）
- `get random()` min max + `[last result] <= n` 分段（906）／`check random()` a b 概率门（669）
- `[hp] % >= n`（764）、`[target state] n`（580，8=倒地）、`[target distance]`（314）、`[hit]`（206）
- `[my active status]`、`[cube item number]`、`[import ai]` `外部.ai`（738，**import 链可做 HP 分段换技能表**——fuhrman Action→Action1/2/3 实证）
- set 系（`[set target position]` 等）= 副作用指令非条件

**典型模式**：优先级 fallback 链（0 号 CD→试 1 号→…）+ 概率门 + 距离环（近环近战/远环远程）；heroaction.ai=英雄难度专用替换（多解锁技能）。

## 无 [ai pattern] 的旧怪靠什么

- `[intelligence]` 三参数 = 攻击概率 / 移动概率 / 确认间隔 ms（=doc 10338 明文）——底层概率引擎，与 ai pattern **不互斥**（1,262 个新怪两者都有：intelligence 是底盘，.ai 是覆盖层）；
- `[attack kind]` 12 列 × 行与攻击动作 1:1 对齐（808 文件验证）：

| 列 | 含义 | 证据 |
|---|---|---|
| 1 | 释放概率 %（100=CD 好必放；**0=蓄力/起手动作，由脚本或前一动作接续触发**，83% 此类行是起手动） | =doc 11735 + 分布 |
| 2/3 | 概率类（85%/68% 为 0），疑受击反击/倒地追击 | 分布推断 |
| 4 | 0/1 布尔，疑攻击中追踪目标 | 推断 |
| 5 | **攻击触发距离 px**（远程中位 300 / 近战 100） | 强证据 |
| 6-12 | 距离/速度类；(9,11) 与 (10,12) 两对完全相同，疑攻击前后移动参数 | 推断 |

- `[think term]`（中位 500ms）AI 思考周期、`[warlike]` 好战权重（boss~80/小怪 20-30）、`[attack delay]` 前摇、`[keep distance with target]`、`[level of difficulty]`（低难度怪更懒：概率补正+思考间隔补正，推断）。

**驱动模型**：难度表(tbl)×per-mob 补正 → warlike/intelligence 基线 → 每 think term 跑一次决策树 → 命中动作查 cooltime/attack kind → 播 .act → 按对齐的 .atk 结算。

## 坑

1. `.ai` 是 CRLF 文本但 Read 工具按扩展名误判二进制——用 bash cat 读；
2. `[cooltime]`（2,533 文件）与 `[attack kind]`（834）近似互斥（仅 20 共存）——**新怪二选一**；
3. michael 式 cooltime "30001=3000ms+作者自标 idx" 粘连写法——按数值解析即可；
4. `[sit action]`=起身（不是坐下）；DestinationSelect 返回码官方义未知（用例反推）。

## 相关

[[monster-dir]] · [[monster-act]] · [[modify-monster]] · [[add-monster]]

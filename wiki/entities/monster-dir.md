---
title: 实体：monster/ 怪物目录与 .mob
category: entity
tags: [怪物, mob, 目录, tbl]
sources: ["pvfDoc/pvfDoc/【03】怪物修改/10338_怪物全面解读.json",
         "pvfDoc/pvfDoc/【03】怪物修改/11739_怪物属性修改教程(怪物文件整体翻译).json",
         "pvfDoc/pvfDoc/【03】怪物修改/10342_monsterapcdifficultybonus全数据解析.json",
         "pvfDoc/pvfDoc/【03】怪物修改/12061_全局怪物难度解析.json",
         "pvfDoc/pvfDoc/【03】怪物修改/11828_修改怪物每等级对应的属性.json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 实体：monster/ 怪物目录与 .mob

`monster/`（约 11.8 万文件）：.mob 3,459 主定义 + .ani 69,481 动画 + .act 25,352 动作 + .ai 8,779 决策树 + .atk 5,351 命中判定 + .ptl 3,593 投掷物 + .als 1,527 特效挂载。

## 目录组织

- 家族目录（旧怪）：`goblin/ tau/ nugol/` —— 家族内固定布局 `*.mob + animation_*/ + action_/ + attackinfo/ + ai/ + particle/`；
- 资料片目录（新怪）：`newmonsters/<资料片>/<地下城>/<怪名>/`，平铺 `action/ attackinfo/ ai/ animation/`；另有 `110levelnormaldungeon/ act8/ bosstower/ championtower/`、第三方 `ch_* js60_*`；
- 同目录可放**多形态**（如 fuhrman 本体 + `_named` 演出壳，共用资产但 .mob 分开注册）。

## 文件类型职责

| 后缀 | 职责 |
|---|---|
| .mob | 属性/等级/抗性/**动作索引表/AI 引用**/掉落/台词 —— 中枢 |
| .act | 动作时间轴：MOTION + TRIGGER×BEHAVIOR（见 [[monster-act]]） |
| .atk | 单次命中判定（见 [[monster-atk]]） |
| .ai | 决策树：条件 → return 动作索引（见 [[monster-ai]]） |
| .ptl | 投掷物/粒子运动学（move type/重力/landing type，可引 passiveobject 索引） |
| .als | 与 .ani 成对的帧特效清单（`[use animation]`+`[none effect add]`） |
| .lua | 人形怪 AI（character/，UTF-16 LE） |
| .tbl | 参数表（见下） |

## .mob 关键字段（节选，全段清单查源教程 10338/11739）

| 段 | 含义 |
|---|---|
| `[ability category]` | 六维倍率：`` `[HP MAX]` `*` 1300``；`*`=乘基础表、`+`=固定加成 |
| `[ability table]` 0/1/2 | 选基础表：0=monsterbaseparameter / 1=summon / 2=warroom |
| `[base table]` | 新式：直引家族私有 tbl 路径 |
| `[category]` | 种族/护甲/行为标签（human/undead/devil…、melee/range、fixture 定身物等）——**机制标签直接写在类别里**（如 `[non getup invincible]`） |
| 20 种 `[* resistance]` min~max | 全异常/属性抗性；**负抗=设计上可被控**，200≈团本免控 |
| `[weight]`（单位 10g）/[hit recovery]（ms，0=2 秒硬直）/[floating height]（≥1 浮空） | 打感三件套 |
| `[level]` / `[lie frame]` 5 值（浮空/倒地帧） | 显示等级 / 倒地表现 |
| 动作槽 | 新式 `[waiting/move/sit(=起身!)/damage 1/2/down/overturn action]` + `[attack action]` 列表 + `[proc action]`（常驻状态机）`[etc action]`（CUSTOM n）`[last action]`（谢幕）`[come in action]`；旧式同名 `[xx motion]`→.ani |
| `[attack info]` | .atk 路径列表，**与 [attack action] 逐行对齐，空行=无判定** |
| `[cooltime]` | 每攻击动作独立 CD（ms），AI 用 `is the skill in cooltime()` 查 |
| `[attack kind]` 12 列 | 新旧近似互斥的 CD/概率体系（列 1=释放概率%、列 5=触发距离 px 强证据，其余未全解，见 [[monster-ai]]） |
| `[item]` / champion/death tower 系掉落段 | 掉落 |
| `[event super on scr]` | HP 门槛事件霸体（如 80/60/40/20% 各 5 秒 unbreakable+blast） |
| `[name]` | 本版本直写中文 |

⚠️ **同 section 重复出现时后值覆盖前值**（fuhrman.mob 实证）。

## 数值三层模型

```
基础表（monsterbaseparameter.tbl 等，按等级查六维）
  × [ability category] 倍率
  × 难度表（monsterapcdifficultybonus.tbl：普通/冒险/勇士/王者/英雄 各给
    好战+/视野+/攻击几率+/回避/HP 比率/攻速/异常抗性/攻防…）
  （.dgn 可用 [monsterapc diff table] 换本包难度表覆盖全局）
```

## 注册与引用

- `monster/monster.lst`（3,120 条，两行 ID 在前）——**地图只写 ID 不写路径**，改文件名/移动目录必须同步 lst；
- 同一 .mob 可注册多 ID（如 atswordman=56482+64839）；存在日期式 ID（活动怪）；
- **341 个孤儿 .mob** 未注册（本地与封包一致，死内容/样板）；
- **悬空引用实例**：kingnugol.mob 的 [passive object filename] 指向不存在的 .ptl，靠 [passive object index] 数字索引兜底——改投掷物别只信路径。

## 相关

[[monster-act]] · [[monster-ai]] · [[monster-atk]] · [[dungeon-chain]] · 操作：[[add-monster]] / [[modify-monster]]

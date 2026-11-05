---
title: 实体：.atk 攻击判定文件
category: entity
tags: [atk, 判定, 怪物]
sources: ["pvfDoc/pvfDoc/【03】怪物修改/10338_怪物全面解读.json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 实体：.atk 攻击判定文件

单次命中判定的数据文件（5,351 个）。**对齐规则**：`.mob [attack info]` 第 i 行 ↔ 第 i 个攻击动作（[attack action]/[action list]/[attack motion]），空行=该动作无独立判定；.act 内 `ATTACKRECT RESET` 只管时机，**框体与伤害都在 .atk**。`[etc attack info]` **不**与 [etc action] 对位（独立引用）。

## 字段表（按 5,351 文件 section 普查，括号=出现数）

| 组 | 字段 | 说明 |
|---|---|---|
| 攻击本体 | `[attack type]`→`[physic]`(4758)/`[magic]`(511) | 伤害类型 |
| | `[damage bonus]`(3907) | 伤害加成 %（可负，如 teleport.atk -30） |
| | `[absolute damage]`(432) | **真值直伤**（注意：多数 Boss 此处是占位如 12345，真伤走 .mob ability category×基础表；APC 直伤型写真值如 1000） |
| | `[weapon damage apply]`(5163) | 武器伤害适用 |
| 对象 | `[attack enemy]`/`[attack friend]`/`[attack myself]` | 打谁 |
| | `[max target hit number]` | 多段上限 |
| 反应 | `[damage reaction]`→`[damage]/[down]/[blow]/[hit down]/[hit horizon]/[hit lift up]/[bounce]/[none]` | 被打者的反应动作 |
| 位移 | `[push aside]`（击退）/`[lift up]`（浮空）/`[knuck back]` | 击退/挑空力度 |
| 表现 | `[hit wav]`、`[hit info]`→`[cut]/[stuck]/[blood]/[no blood]` | 音效/打击感（`[stuck] 负值`=强造玩家硬直，APC 常用） |
| 属性 | `[elemental property]`→`[no element]/fire/water/dark/light element` | 攻击属性 |
| 异常 | `[active status]`(1216)→`[stun]/[poison]/[burn]/[bleeding]/[freeze]/[lightning]/[curse]/[confuse]/[blind]/[slow]/[weakness]/[stone]` | 参数行如 stun `50 0 3000` = 概率50%/等级0/3000ms（=doc 口径：概率/等级/持续/间隔，火伤 burn 7 参） |
| 特殊 | `[ignore super armor]`/`[ignore defense]`/`[HP drain]`/`[armor break]`/`[force hit stun time]` | 破霸体/破防/吸血等 |

## 修改要点

- **调伤害**：先判断该怪是"倍率型"（absolute 是占位 → 改 .mob `[ability category]` 或 `[damage bonus]`）还是"直伤型"（改 `[absolute damage]`）——看同族用例与数值合理性；
- **加控制**：`[active status]` 加段；**改反应**：换 `[damage reaction]` 分支；
- fuhrman 实例：hit.atk=bonus 50%+50% 眩晕 3s；powerpunch.atk=absolute 2000+倒地+击退 600/浮空 500；teleport.atk=bonus -30%（多段弱伤害走量）。

## 相关

[[monster-dir]] · [[monster-act]]（ATTACKRECT 时机）· [[modify-monster]]

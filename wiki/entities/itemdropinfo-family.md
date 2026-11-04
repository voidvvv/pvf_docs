---
title: 掉率三件套：itemdropinfo 系列
category: entity
tags: [掉率, itemdropinfo, etc]
sources: ["物品掉率文件修改说明.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 掉率三件套：itemdropinfo 系列

`etc/` 下三个怪物掉率文件，共享相同的段落结构，差异在控制方式：

| 文件 | 作用 | 稀有度控制 |
|------|------|------------|
| `itemdropinfo_monster_hell.etc` | 深渊副本（困难 / 非常困难） | `[basis of rarity dicision]`（**带头部档位计数值 2**）+ `[dungeon difficulty drop prob]` |
| `itemdropinfo_monseter.etc` | 普通副本普通怪 | `[basis of rarity dicision]`（无头部计数，直接 4 行）+ `[drop prob]` + 多组 bonusrate |
| `itemdropinfo_monseter_extra.etc` | 普通副本精英 / Named 怪 | 同普通文件 |

> ⚠️ 文件名 `monseter` 是**原版拼写**，不可"纠正"为 `monster`，否则游戏无法识别（见 [[naming-traps]]）。

## 核心段落

### `[basis of rarity dicision]` 稀有度判定

每行 6 列：`[白阈值][蓝阈值][紫阈值][粉阈值][史诗上限][难度码]`，算法见 [[probability-basis]]。

**当前实测爆率**：

- 深渊·困难与非常困难完全相同：白 40% / 蓝 30% / 紫 20% / 粉 6% / 史诗 4%（非常困难档每阈值 +1，差值不变，仅避免随机区间重叠，**无实质影响**）。
- 普通 / 精英各 4 个掉落档位，**史诗档几乎为 0**（末档阈值顶满 1000000）——普通副本不爆史诗，符合游戏设定。精英档位 1、2 数值优于普通，档位 3、4 与普通完全相同。
- 档位与难度 / 怪物类别的映射需结合 `[dungeon difficulty drop bonusrate]` 与 `[monster type drop bonusrate]` 联合确认。

### `[item drop ref table]` 物品等级映射

每行三列 `[地图等级] [减去的下限] [加上去的上限]`：

```
物品等级下限 = 地图等级 − 第2列
物品等级上限 = 地图等级 + 第3列
```

例：`50 8 3` → 42~53 级。深渊文件低等级段范围宽（`1 0 15` → 1~16），普通 / 精英紧贴地图等级（`5 4 0` → 1~5）。

### `[drop prob]` 等级分段掉率（仅普通 / 精英）

`[drop prob count]` = 7 → 每 7 个数字一组，实际文件中 7 组共 49 个数字**写在同一行**（Tab 分隔）。每组 7 列：

| 列 | 含义 |
|----|------|
| 1~2 | 等级下限 / 上限 |
| 3 | **金币掉落概率**（÷1,000,000，`986` = 0.0986%） |
| 4 | 掉落类别 B（待确认，疑似材料 / 消耗品） |
| 5 | **装备掉落概率**（÷1,000,000） |
| 6~7 | 掉落类别 C / D（待确认；精英文件出现非 0 值 `15 30`） |

> 🔑 曾有"‰ 千分比"的误读，已纠正：基数 = 1,000,000（见 [[probability-basis]]）。普通文件覆盖 1~99 级，精英延伸到 200 级。

### 倍率段（仅普通 / 精英）

**最终掉率 = 基础掉率 × 组队倍率 × 难度倍率 × 怪物类型倍率**

| 段落 | 含义 | 峰值示例 |
|------|------|----------|
| `[party member drop bonusrate]` | 组队人数（1~4 人） | 4 人 ×1.6 / ×2.0 |
| `[dungeon difficulty drop bonusrate]` | 副本难度 | 王者 ×2.2 |
| `[monster type drop bonusrate]` | 怪物类型 | Boss ×6.6（普通）/ ×4.0（精英） |

### 深渊专属与尾部段

- 深渊：`[dungeon difficulty drop prob]`（2 组基础掉落数值）、`[drop prob count]` = 2。
- 普通 / 精英尾部：`[first boss/named mob hunting]`（首杀标记 `1 1`）、`[condition rate]`（如 `90 10`）、`[gold quantity]` / `[gold volume]`（金币档位与数量）。

## 修改要点

阈值严格递增、末档 = 1,000,000、末列难度码勿改、Tab 分隔保持。实操示例（提史诗爆率、让普通本出史诗、扩等级范围）见 [[modify-drop-rate]]。

## 相关

- 世界掉落侧：[[worlddrop-etc]]（触发时与怪物掉率并行判定）
- 来源摘要：[[src-drop-rate-doc]]

---
title: 勇者纹章道具族（100000101~110）
category: entity
tags: [纹章, 道具, stackable, crest]
sources: ["快捷栏装备系统与Appendage使用指南.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 勇者纹章道具族（100000101~110）

快捷栏装备的具体道具族：道具代码 `100000101` ~ `100000110`，文件位于 `stackable/crest/<道具码>.stk` + `equipment/ap/ap_<道具码>.nut`。

## 道具与效果对照

| 道具码 | 效果（onStart 实测值） |
|--------|------------------------|
| `100000101` | 物攻 / 物防 / 魔攻 / 魔防 各 **+10**，命中 −1（`ap_100000101.nut`） |
| `100000110` | **+500 全属性大礼包**：物攻 / 魔攻 / 物防 / 魔防 / 装备物攻 / 装备魔攻 / 独立物攻 / 独立魔攻 / 全属强 / 物暴伤 / 魔暴伤 全部 +500（`ap_100000110.nut`，常用作多属性 ap 模板） |

> 102~108 为同模板不同数值的纹章（完整清单以 `stackable/crest/` 实际文件为准）；110 段曾有结构写坏问题（见 [[quickbar-equipment-system]]）。

## .stk 道具文件要点

- 说明文字写"请放入快捷栏位置，副本内生效"；`[flavor text]` 写 `快捷栏装备`；
- **不需要 `[append]` 段**——效果 100% 由 sqr 扫描挂载（机制见 [[quickbar-equipment-system]]）。

## 相关

- 新增纹章步骤：[[add-quickbar-equipment]]
- 属性修改 API：[[appendage]]（CHANGE_STATUS_TYPE_* 常量表）
- 来源摘要：[[src-quickbar-guide]]

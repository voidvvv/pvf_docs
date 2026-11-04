---
title: load_state 脚本注册机制
category: concept
tags: [nut, loadstate, 注册]
sources: ["input/my_inclusion.md", "快捷栏装备系统与Appendage使用指南.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# load_state 脚本注册机制

每个职业有一个 `{职业}_load_state.nut`（位于 `sqr/character/`），是 nut 脚本的**注册入口**；总入口为 `sqr/loadstate.nut`。未被注册的脚本不会被引擎加载和调用。

## 三个注册原语

```squirrel
// 1. 加载脚本文件（header / 公共函数等）
IRDSQRCharacter.pushScriptFiles("Character/ATMage/atmage_header.nut");

// 2. 注册技能状态：职业枚举ID, 脚本路径, 状态名, STATE常量, SKILL常量
//    SKILL 参数为 -1 表示不绑定特定技能（如普通攻击、站立等基础状态）
IRDSQRCharacter.pushState(ENUM_CHARACTERJOB_AT_MAGE, "Character/ATMage/FireRoad/fire_road.nut",
                          "FireRoad", STATE_FIRE_ROAD, SKILL_FIRE_ROAD);

// 3. 注册被动对象：脚本路径 + 唯一ID（必须与 passiveobject.lst 中 .obj 的 ID 对应）
IRDSQRCharacter.pushPassiveObj("Character/ATMage/FireRoad/po_fire_road.nut", 24212);
// 同一脚本可注册多个 ID（同一技能创建多个同类被动对象）
```

另有 `sq_RunScript("路径")` 运行不绑定角色的独立脚本。

## 加载顺序（loadstate.nut，源工作区实测）

```
loadstate.nut
  L7   dnf_enum_header.nut
  L8   common.nut                       （setState_AllGrowJob、IsInterval 等）
  L10  Character/common_load_state.nut  （common_header / common_common / burster）
  L14  js60_qq506807329/js60_506807329_common.nut
  L19  Character/priest_load_state.nut
         L129 → character/newpriest/rangebuff/rangebuff.nut  （useRangeBuff，见 [[procappend-priest-bug]]）
  L31  apjh/lue.nut                     （AllGrowJob 系列钩子）
  L43  equipment/equipment_main.nut     （各 equipment_<job>.nut，含 procAppend_<job>）
  L47  common_class.nut / common_function.nut / ui 等
```

> 关键结论：所有 `<job>_load_state.nut` 都在 `equipment_main.nut`（L43）**之前**加载——`procAppend_<job>` 运行时，其前缀调用的函数（`procAppend_AllGrowJob`、`useRangeBuff` 等）早已全局可见。因此"前缀函数未定义"不是问题来源；出问题通常是**运行时**错误（null 解引用等）。

## 注册新内容的检查单

新增技能 / 被动对象 / 独立脚本时：

1. 脚本文件放对目录（命名规则见 [[nut-scripting]]）；
2. `pushScriptFiles` / `pushState` / `pushPassiveObj` 挂到对应 load_state；
3. 被动对象还需在 `passiveobject.lst` 注册 `.obj`（见 [[passive-object]]）；appendage 若走 `.apd` 路线需在 `appendage.lst` 注册（见 [[appendage]]）；
4. 常量（STATE_* / SKILL_*）在 header 中定义；
5. `import_file` + `save_as_pvf`。

## 相关

- 职业枚举：`ENUM_CHARACTERJOB_*`（0=SWORDMAN、1=FIGHTER…，详见 `input/raw/nut_note.txt`，索引：[[input-raw-references]]）
- 来源摘要：[[src-my-inclusion]]、[[src-quickbar-guide]]

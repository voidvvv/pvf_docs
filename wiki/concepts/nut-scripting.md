---
title: nut（Squirrel）脚本体系
category: concept
tags: [nut, squirrel, sqr, 脚本]
sources: ["input/my_inclusion.md", "快捷栏装备系统与Appendage使用指南.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# nut（Squirrel）脚本体系

`.nut` 是 DNF PVF 中的 Squirrel 脚本，客户端与服务端共同解析，通过**钩子函数**扩展游戏逻辑（新增常驻状态、改技能形态、甚至新增技能）。全部存放在 PVF 的 `sqr/` 目录（本地 `huanjing/sqr/`，路径映射见 [[huanjing-path-mapping]]）。

## sqr 目录结构（按职能 / 角色）

| 子目录 | 作用 |
|--------|------|
| `character/`（~1768 文件） | 角色脚本：`common/` 公共 + 各职业（swordman / fighter / gunner / mage / priest / thief 及 `at*` 转职） |
| `map/` `monster/` | 地图 / 怪物脚本 |
| `equipment/` | 装备逻辑（快捷栏装备、纹章等，见 [[quickbar-equipment-system]]） |
| `passiveobject/` | 被动物物脚本（见 [[passive-object]]） |
| `appendage/` | appendage 脚本（见 [[appendage]]） |
| `ui/` `functions/` `custom/` | UI / 公共函数 / 自定义内容 |
| `js60_qq506807329/` `apjh/` `qq358332886_monster_appendage/` `unclebang_shared_passive_object/` | 第三方模组命名空间（见下） |

顶层全局脚本：`common.nut`、`common_function.nut`、`dnf_enum_header.nut`、`init_character.nut`、`loadstate.nut`（加载入口，见 [[load-state]]）。

## 每个职业目录的典型结构

```
sqr/character/{职业}/
├── {职业}_header.nut      # 常量：STATE_* / SKILL_* / CUSTOM_ANI_*
├── {职业}_common.nut      # 公共函数
├── {技能名}/              # 每技能一目录：{技能名}.nut + po_*.nut + ap_*.nut
├── appendage/ap_*.nut
└── …
sqr/character/{职业}_load_state.nut   # 注册入口（在 character/ 下）
```

## 文件类型与命名规则

| 类型 | 命名 | 说明 |
|------|------|------|
| Header | `{职业}_header.nut` | 定义 STATE_* / SKILL_* / CUSTOM_ANI_* / 元素等常量，加载后全职业脚本可见 |
| Load State | `{职业}_load_state.nut` | 注册入口（pushScriptFiles / pushState / pushPassiveObj） |
| 技能状态 | `{技能名}.nut` | 技能行为（状态切换、动画、创建被动对象） |
| 被动对象 | `po_{名称}.nut` | 弹道 / 特效区域行为 |
| Appendage | `ap_{名称}.nut` | 状态效果行为 |
| 被动技能 | `passive_skill_{职业}.nut` | 被动技能如何创建 / 附加 appendage |

## 技能状态脚本的钩子函数

| 函数名格式 | 说明 |
|-----------|------|
| `checkExecutableSkill_{名称}(obj)` | 检测技能输入 |
| `checkCommandEnable_{名称}(obj)` | 当前状态是否允许该指令 |
| `onSetState_{名称}(obj, state, datas, isResetTimer)` | 进入状态：设子状态、播动画 |
| `onEndCurrentAni_{名称}(obj)` | 动画播完：切换子状态 / 回站立 |
| `onKeyFrameFlag_{名称}(obj, flagIndex)` | 关键帧触发：常用于创建被动对象 |
| `onEndState_{名称}(obj, new_state)` | 离开状态：清理 |

典型流程：按键 → `checkExecutableSkill` → 切 STATE → `onSetState` 播施法动画 → `onEndCurrentAni` 切释放阶段 → `onKeyFrameFlag` 处 `sq_SendCreatePassiveObjectPacket` 创建被动对象 → 回 STATE_STAND。

## 本服模组命名空间（86幻境叠加多个模组）

| 命名空间 | 含义 |
|---|---|
| `js60_qq506807329/` | 模组作者 js60 的共享被动体与职业扩展 |
| `apjh/` | 公共钩子层（`procAppend_AllGrowJob`、`duegonUsableItem` 等） |
| `qq358332886_monster_appendage/` | 怪物 appendage |
| `unclebang_shared_passive_object/` | 共享被动体 |
| `custom/content/` | 自定义内容（训练房、新技能键、段位系统等） |

## ⚠️ 每帧函数异常即静默中断

Squirrel 异常会被 DNF 引擎在 C++/Squirrel 边界**静默捕获**：不崩溃、不报错，但**本帧该函数剩余代码全部跳过**。写在每帧钩子（`procAppend_*` / `drawMainCustomUI_*`）末尾的逻辑可能因此永不执行——典型案例见 [[procappend-priest-bug]]。防范：每帧函数内所有解引用先做 null 保护。

## 相关

- 注册机制：[[load-state]]　·　状态载体：[[appendage]]　·　弹道系统：[[passive-object]]
- 修改/新增脚本流程：[[modify-nut-script]]
- 来源摘要：[[src-my-inclusion]]、[[src-quickbar-guide]]

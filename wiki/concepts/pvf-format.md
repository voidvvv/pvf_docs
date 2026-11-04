---
title: PVF 封包与文件类型体系
category: concept
tags: [pvf, 格式, 基础]
sources: ["AGENTS.md", "input/my_inclusion.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# PVF 封包与文件类型体系

**PVF**（`Script.pvf`）是 DNF 的数据封包格式，客户端与服务端共同读取。工作区的最终目标：解析、修改、扩展包内数据表与脚本，再编译回 PVF 供游戏加载。**任何修改都要 `save_as_pvf` 重新打包才生效。**

## 文件类型速查

| 后缀 | 类型 | 位置 | 说明 |
|------|------|------|------|
| `.etc` / `.tbl` | 数据表 | `etc/` | `[section]` 分段、Tab 分隔的文本表 → [[etc-table-format]] |
| `.nut` | Squirrel 脚本 | `sqr/` | 钩子函数扩展游戏逻辑 → [[nut-scripting]] |
| `.skl` | 技能定义 | `skill/<职业>/` | 技能属性、等级数据 → [[analyze-skill]] |
| `.apd` | appendage 定义 | `appendage/` | 声明式状态效果 → [[appendage]] |
| `.obj` | 被动物体定义 | `passiveobject/` | 视觉与碰撞属性，配合同名 `.nut` → [[passive-object]] |
| `.stk` | 可堆叠道具 | `stackable/` | 消耗品 / 礼盒 / 纹章等 → [[src-gift-box]]、[[crest-items]] |
| `.equ` | 装备定义 | `equipment/` | 装备属性（含 rarity/等级/价格字段）→ [[equ-format]] |
| `.lst` | 索引表 | 各目录 | 数字 ID → 文件路径 / 名称 映射 → 下文 |
| `.chr` / `.ani` / `.img` / `.lay` | 角色 / 动画 / 图片 / 层级 | `character/`、`equipment/` | 渲染链 → [[src-atswordman-render]] |

## .lst 索引机制

`.lst` 是"数字 ID → 文件路径"的映射索引，遍布各资源目录：

| LST | 作用 |
|-----|------|
| `equipment/equipment.lst` | 装备代码 → `.equ` |
| `stackable/stackable.lst` | 消耗品代码 → `.stk` |
| `skill/<职业>skill.lst` | 技能代码 → `.skl`（各职业一份） |
| `passiveobject/passiveobject.lst` | 被动物体 ID → `.obj` |
| `appendage/appendage.lst` | appendage ID → `.apd` |

- 新增 appendage / 被动对象 / 技能 / 道具时，**必须在对应 `.lst` 注册 ID**；
- 物品代码反查用 pvfutility 的 `item_code_to_file_info`（见 [[pvfutility-mcp]]）；
- 各职业技能 LST 对照表见 [[analyze-skill]]。

## PVF 根目录布局

`sqr/`（脚本）、`etc/`（数据表）之外：`appendage/`、`passiveobject/`、`character/`、`equipment/`、`skill/`、`monster/`、`map/`、`stackable/`、`npc/` 等。本地解包副本在 `huanjing/` 下（见 [[huanjing-path-mapping]]）。

## 修改通用流程

读原文（MCP `get_file_content`）→ 确认结构 → 编辑 → `import_file` 写回 → `save_as_pvf`。完整步骤见 [[pvfutility-mcp]] 与 [[modify-drop-rate]]。

## 相关

- 来源摘要：[[src-agents-md]]、[[src-my-inclusion]]

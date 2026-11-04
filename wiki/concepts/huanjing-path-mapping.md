---
title: 目录映射：huanjing/ 前缀 ⇄ DNF 内路径
category: concept
tags: [路径, 目录结构, 基础]
sources: ["物品掉率文件修改说明.md", "AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 目录映射：huanjing/ 前缀 ⇄ DNF 内路径

源工作区（`D:\pvf`）是 DNF「86幻境」版本的 PVF 工作区。`huanjing/` 只是**版本名**（本地已解包的 PVF 内容），其内部一级目录才是真正的 DNF 游戏目录。

## 映射规则

| 本项目路径 | 等价 DNF 内路径 | 内容 |
|-----------|----------------|------|
| `huanjing/etc/` | `etc/` | 数据配置表（200 项：`.etc` / `.tbl` / `.lst`） |
| `huanjing/sqr/` | `sqr/` | Squirrel 脚本（`.nut`，按职能 / 职业分子目录） |

**换算规则：真实 DNF 路径 = 去掉 `huanjing/` 前缀。**

| 本地写法 | 真实 DNF 路径 |
|----------|---------------|
| `huanjing/etc/worlddrop.etc` | `etc/worlddrop.etc` |
| `huanjing/sqr/equipment/equipment_swordman.nut` | `sqr/equipment/equipment_swordman.nut` |

## 关键约定

- **pvfutility MCP 调用一律传 DNF 内路径**（不带 `huanjing/` 前缀）。
- 两套数据源的关系：
  - `huanjing/`（本地工作副本）→ 可用 Read/Edit 直接改；**已被 gitignore，不入库**；
  - PVF 封包（MCP 操作对象，游戏真正读取）→ 修改用 `import_file` 写回。
  - 两条路径最终都必须 `save_as_pvf` 重新打包才生效（见 [[pvfutility-mcp]]）。
- 引用原版文件名时保持原拼写，不可"纠正"（见 [[naming-traps]]）。

## PVF 根目录速查

`sqr/`（脚本）、`etc/`（数据表）之外，PVF 封包内还有：`appendage/`（.apd）、`passiveobject/`（.obj）、`character/`、`equipment/`、`skill/`、`monster/`、`map/`、`stackable/`、`npc/` 等，详见 [[pvf-format]]。

## 相关

- 文件类型体系：[[pvf-format]]
- 数据表格式：[[etc-table-format]]
- 脚本体系：[[nut-scripting]]
- 来源摘要：[[src-drop-rate-doc]]、[[src-agents-md]]

---
title: equipment/equipment.lst 装备索引
category: entity
tags: [lst, equipment, 物品代码]
sources: ["AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# equipment/equipment.lst 装备索引

装备目录的 `.lst` 索引：**物品代码 ↔ .equ 路径** 映射。实测（2026-11-04，86幻境导出副本）：112,243 条映射，对应 116,094 个 .equ 文件（含少量未注册文件）。

## 文件格式：两行一条记录

与 `.obj` 的 LST 单行格式（`ID -> path "名称"`，见 [[passive-object]]）**不同**，equipment.lst 是两行一条：

```
10000
`character/common/jacket/cloth/sht_ragcloth.equ`
10001
`character/common/...`
```

- 第 1 行：纯数字物品代码；
- 第 2 行：反引号包裹的路径；
- **路径相对 `equipment/` 目录**（不带 `equipment/` 前缀），比对时需剥前缀且大小写不敏感；
- 文件头有 `#PVF_File` 标记行。

### 本地解析要点（状态机）

逐行扫描：遇纯数字行记为待配代码，遇 `` `….equ` `` 行配对成记录；其他行忽略。不要用单行正则（会 0 命中——实测踩坑）。

## ⚠️ 勿用 MCP 整读此文件

`get_lst_file_info("equipment/equipment.lst")` 因 11.5 万条超载导致 **MCP 连接中断**（Connection closed，2026-11-04 实测）。替代方案：用 pvfutility 把 `equipment/` 导出到本地 `huanjing/`（见 [[huanjing-path-mapping]]）后本地解析，详见 [[pvfutility-mcp]] 实测语义节。

## 用途

- 物品代码 → 文件：`item_code_to_file_info`（MCP 单条反查，安全）或本地 LST 解析；
- 输出装备清单时补物品代码列（2026-11-04 扫描中 3,628 件命中里 284 件未匹配到 LST，多为 `common/odd`、`common/magicstone` 目录道具——未注册或路径别名，原因未深究）。

## 相关

- .equ 格式：[[equ-format]]　·　LST 机制总览：[[pvf-format]]
- 来源摘要：[[src-agents-md]]

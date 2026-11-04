---
title: .etc 数据表格式规范
category: concept
tags: [etc, 格式, 数据表]
sources: ["AGENTS.md", "物品掉率文件修改说明.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# .etc 数据表格式规范

`.etc`（及 `.tbl`）是 PVF 中的纯文本数据表，位于 `etc/`（本地路径 `huanjing/etc/`，见 [[huanjing-path-mapping]]）。

## 基本语法

```
[section name]          ← 段落名，用 [] 包裹
    400000  700000  900000   ← 段内数据行，数值用 Tab 分隔
[/section name]         ← 部分段落有闭合标记（如 [world drop]）
```

- **段落**：`[xxx]` 分段；部分段落（`[etc]`、`[string data]`、`[world drop]` 等）带 `[/xxx]` 闭合。
- **分隔符**：段内数值一律 **Tab 分隔**。
- **组内计数**：某些段落（如 `[drop prob count]`）给出"每 N 个数字一组"的列数，引擎按它把后续数据切片。

## 修改铁律

1. **保留原有空白与缩进**——修改时只动数值，不动结构；
2. **保持文件名不变**——含原版拼写（如 `monseter`），见 [[naming-traps]]；
3. 数值含义不确定时查 `pvfDoc/`（见 [[pvfdoc-library]]）或对照实际文件，**不臆造字段含义**；
4. 改完必须 `import_file` 写回 + `save_as_pvf` 重新打包（见 [[pvfutility-mcp]]）。

## 典型 .etc 文件

| 文件 | 作用 | 详见 |
|------|------|------|
| `etc/itemdropinfo_monster_hell.etc` | 深渊怪物掉率 | [[itemdropinfo-family]] |
| `etc/itemdropinfo_monseter.etc` | 普通怪物掉率 | [[itemdropinfo-family]] |
| `etc/itemdropinfo_monseter_extra.etc` | 精英怪物掉率 | [[itemdropinfo-family]] |
| `etc/worlddrop.etc` | 世界掉落（1~200 级分块） | [[worlddrop-etc]] |

概率数值换算见 [[probability-basis]]。

## 相关

- 修改流程：[[modify-drop-rate]]　·　格式对比：[[pvf-format]]
- 来源摘要：[[src-agents-md]]、[[src-drop-rate-doc]]

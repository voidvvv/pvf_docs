---
title: 工具：pvfutility MCP
category: entity
tags: [工具, mcp, pvf]
sources: ["AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 工具：pvfutility MCP

读取 / 修改 PVF 内容的**首选工具**（比手写文件解析可靠）。操作对象是**已载入的 PVF 封包**（游戏真正读取的内容），当前载入路径可用 `get_pvf_pack_file_path` 查询（源工作区为 86幻境客户端的 `Script.pvf`）。

## 接口速查

| 工具 | 用途 |
|------|------|
| `get_pvf_pack_file_path` | 查询当前载入的 PVF 路径 |
| `get_pvf_root_directory` | 列出 PVF 根目录 |
| `get_file_list` / `get_all_lst_file_list` | 列目录 / 列所有 LST |
| `get_file_content` | 读单个文件（编码 TW/CN/KR/JP/UTF8） |
| `get_file_contents_batch` | 批量读 |
| `search_pvf` | 按关键词 / 正则搜索（可限定目录） |
| `get_lst_file_info` | 读 LST 索引（ID → 文件 / 名称） |
| `item_code_to_file_info` / `get_item_info` | 物品代码 → 文件 / 物品名 |
| `import_file` / `import_files_batch` | 写回 / 覆盖文件内容到 PVF |
| `save_as_pvf` | **另存为新 PVF 封包（重新打包）** |

## 关键约定

- **路径参数传 DNF 内路径**（不带 `huanjing/` 前缀），如 `etc/worlddrop.etc`、`sqr/equipment/equipment_swordman.nut`——映射关系见 [[huanjing-path-mapping]]。
- 与本地副本的分工：读权威内容 → MCP；本地增量编辑 → 改 `huanjing/` 或 MCP 写回；**最终都必须 `save_as_pvf` 才生效**。

## 标准修改流程

```
1. search_pvf / get_file_list 定位
2. get_file_content 读原文（确认结构，绝不臆造字段）
3. 编辑（保持 [section] / Tab 结构，见 [[etc-table-format]]）
4. import_file 写回
5. save_as_pvf 另存为新 PVF → 替换游戏目录的 Script.pvf
```

实例化流程页：[[modify-drop-rate]]（数据表）、[[modify-nut-script]]（脚本）、[[analyze-skill]]（技能）。

## 相关

- 运行时 hook 的互补工具：[[frida-js]]
- 来源摘要：[[src-agents-md]]

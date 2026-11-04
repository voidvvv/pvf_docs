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

## 实测语义与坑（2026-11-04 会话）

**`search_pvf`**（经 ring 目录 2,147 文件对照验证）：

- `search_folder` 生效且精确限定；plain 模式 = **内容子串匹配**（非文件名）；每个 .equ 均含 `[rarity]`（2147/2147 命中）；
- **只在行内匹配**：跨行字面量（`[rarity]\r\n\t6` 真实控制字符）0 命中——不能用 plain 搜索按"下一行的数值"筛文件；
- **regex 模式不可靠**：中文字符串可命中，但 `\[rarity\]`、`min…\]\r\n\t90` 等带转义/跨行的模式全部 0 命中且无报错——**禁止用 regex 模式做结论性筛选**。

**`get_lst_file_info` 大 LST 会压垮连接**：`equipment/equipment.lst`（11.5 万条）→ `MCP error -32000: Connection closed`，且此后本会话对该服务器的调用被持续误路由。**超大索引走本地方案**。

**本地导出替代路径（用户 2026-11-04 实践）**：用 pvfutility 把 `equipment/` 导出到 `huanjing/`（见 [[huanjing-path-mapping]]），本地批量扫描——文件为 UTF-8，39,707 个非时装 .equ 全量解析无压力；LST 两行记录格式的解析方法见 [[equipment-lst]]。⚠️ 但**导出副本 ≠ 封包实况**（个别文件可能被本地手改过）——"无需修改"类跳过判断必须对封包实况验证，教训见 [[pink-price-incident]]。

**直连 HTTP API 批量导入（大规模写操作首选，2026-11-04 实测）**：MCP 壳 `mcp_server.py` 只是 `http://localhost:27000` 的代理。直连 REST 可让**文件内容从磁盘直达、零耗 LLM 上下文**：

| 操作 | 端点 |
|------|------|
| 批量导入 | `POST /Api/PvfUtiltiy/ImportFiles`，body=`[{FilePath, FileContent}]` JSON 数组 |
| 单件导入 | `POST /Api/PvfUtiltiy/ImportFile?filePath=…`，body=文本 |
| 单件读回 | `GET /Api/PvfUtiltiy/GetFileContent?filePath=…&encodingType=UTF8` |
| 另存封包 | `GET /Api/PvfUtiltiy/SaveAsPvfFile?filePath=<urlencoded>` |

实测 40 件/批 × 2,604 件导入仅 1 秒、0 错误；单件读回全量审计 3,628 件仅 26 秒。断点续传模式：进度写 json，重跑只补未完成批次（demo/_work/direct_import.py 范式）。

## 相关

- 运行时 hook 的互补工具：[[frida-js]]
- 来源摘要：[[src-agents-md]]

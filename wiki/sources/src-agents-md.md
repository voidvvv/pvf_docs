---
title: 来源摘要：AGENTS.md（项目说明书）
category: source
tags: [项目, 约定, 工具]
sources: ["AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 来源摘要：AGENTS.md（项目说明书）

> 原文：`AGENTS.md`（源工作区根目录，`CLAUDE.md` 的实际内容）。面向 AI 助手的项目说明书——本 wiki 的 schema 层与它角色互补：AGENTS.md 管"怎么在项目里干活"，wiki 管"沉淀下来的知识"。

## 核心结论（3~5 句）

1. 工作区是 DNF 86幻境 PVF 模组 / 分析区，两套数据源（`huanjing/` 本地副本 vs MCP 载入的 PVF 封包），最终都要 `save_as_pvf` 重新打包；
2. 七大避坑约定：路径映射、概率基数 1,000,000、原版拼写不纠正、.etc Tab/[section] 格式、.nut 钩子约定、LST 注册、gitignore 范围；
3. pvfutility MCP 是首选工具（11 个接口）；技能分析走 `skills/dnf-skill-analyzer`；运行时 hook 走 `frida.js`；
4. 参考资料分三处：`input/my_inclusion.md`（入门）、`pvfDoc/`（19 类教程）、根目录既有分析文档。

## 已生成的 wiki 页

| 页面 | 提取内容 |
|------|----------|
| [[pvf-format]] | 文件类型体系、LST 机制、根目录布局 |
| [[etc-table-format]] | .etc 语法与修改铁律 |
| [[pvfutility-mcp]] | 接口速查、标准修改流程 |
| [[frida-js]] | 运行时 hook 定位与 PVF 的互补 |
| [[huanjing-path-mapping]] | 路径映射（与掉率文档互为来源） |
| [[naming-traps]] | 拼写红线 |
| [[modify-nut-script]] | 脚本修改通用流程 |

## 原文中未搬运、需回原文的细节

- `sqr/` 各子目录的完整文件数统计；
- `pvfDoc` 19 类的原始清单（已并入 [[pvfdoc-library]]，但原始表格在 AGENTS.md §9）；
- §六 常见任务流程的原始表述。

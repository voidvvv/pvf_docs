---
title: 操作日志（log）
category: meta
tags: [日志]
sources: []
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 操作日志（append-only）

> 格式约定见 [[WIKI]]。新条目追加在文件末尾，禁止修改历史条目。

## [2026-11-04] bootstrap | 初始批量 ingest（依据 wiki改造方案.md §9）
- 新增：32 页 —— 根级 4（WIKI / index / log / overview）+ concepts×8 + entities×8 + howto×4 + debugging×2 + sources×6
- 来源：物品掉率文件修改说明.md、AGENTS.md、快捷栏装备系统与Appendage使用指南.md、input/my_inclusion.md（全量 ingest）；gift_box_analysis.md、atswordman_render_analysis.md（仅摘要页）；pvfDoc/（仅导航页）；input/raw/（仅索引页）；skills/dnf-skill-analyzer/SKILL.md（并入 howto/analyze-skill）
- 备注：用户已确认方案四决策点——wiki 放本项目内、全量 31 页、英文 slug、pvfDoc 按需 ingest；实际落地 30 页（原计划 howto/modify-and-repack 并入 pvfutility-mcp 标准流程，未单列）
- 待办：礼盒系统与渲染链仅摘要（见 src-gift-box / src-atswordman-render 的"待深挖"节）；[drop prob] 第 4/6/7 列类别待确认

## [2026-11-04] lint | 首次健康检查
- 检查：断链（33 个链接目标，剔除 WIKI.md 内文档示例 [[slug]] 后 0 断链）/ 孤儿页（0）/ 超 200 行页面（0）/ frontmatter（修复 WIKI、index、log 3 个根级文件补齐）
- 修正：index 与 log 的页数统计 30 → 32（实际 4+8+8+4+2+6=32）
- 备注：礼盒与渲染链仅有摘要页属"已知缺口"而非 lint 问题（见 overview 的知识缺口节）

## [2026-11-04] ingest | .equ 格式 / equipment.lst 格式 / NPC 价格规则（粉装扫描会话产出）
- 新增：entities/equ-format（字段表 + rarity 映射 0~5 双文档证据 + 未证实字段清单 + rarity 6/7 实测警示）、entities/equipment-lst（两行记录格式 + 解析要点 + MCP 整读崩溃警示）、concepts/npc-price（[price] 买入 / [value] 卖出÷5 —— 来源：用户实测告知，暂无 pvfDoc 佐证）
- 更新：entities/pvfutility-mcp（新增"实测语义与坑"：search 行内匹配、regex 模式不可靠、大 LST 崩连接、本地导出替代路径）、concepts/pvf-format（.equ 行链接 equ-format）、sources/src-gift-box（相关链接 npc-price）、overview（知识缺口刷新：.equ 已覆盖，rarity 6/7 待证实）、index（35 页）
- 备注：粉装扫描结果本体（3,628 件）存 demo/（gitignore），不视为 wiki 知识；方法与字段语义已沉淀

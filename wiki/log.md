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

## [2026-11-04] ingest | 粉装售价事故排查（负价溢出 + 导出副本漂移）
- 新增：debugging/pink-price-incident（int32 溢出负价 -89,934 完整数值链；导出副本≠封包实况导致 2 件遗漏；数据源信任层级与全量审计方法论）
- 更新：concepts/npc-price（新增 int32 溢出上限节：[value]≤21,474,836，到手≤4,294,967）、entities/pvfutility-mcp（新增直连 HTTP API 批量导入节：ImportFiles 端点、40件/批实测 2604 件 1 秒、审计 3628 件 26 秒；导出副本≠封包警示）、index（36 页）
- 备注：v1 公式（等级×500,000）因溢出全量废弃，v2 改为固定 [value]=20,000,000（到手 400 万，用户确认）；回滚清单 v2 含封包真实原值（modify_manifest_v2.json）

## [2026-11-04] update | 售价溢出乘数模型修正（第二实测点淘汰 ×100）
- 更新：concepts/npc-price（乘数订正为 ×200 或 ×1000；全模型安全上限 [value]≤2,147,483/到手≤429,496；附 2,500,000 判别探针；留痕曾误判 ×100 的原因）、debugging/pink-price-incident（新增第二轮 −294,967 推演与"多模型同解须判别性实验"方法论教训）
- 修正原因：v2 按错误 ×100 模型设 20,000,000，游戏实测 −294,967 与 ×100 预测 +4,000,000 矛盾；反推除数=1000 确认 ×200/×1000
- 终版：全部 2,609 件固定 [value]=2,000,000（到手 40 万，用户确认）；三次审计 2,609/2,609 全对；回滚表 modify_manifest_v3.json（含封包真实原值）

## [2026-11-05] ingest | NPC 商店知识（用户确认后入库）
- 新增：entities/npc-shop（.shp 格式实测解析：[NPC]/[type]/[message]/[sell item]/[tab name]；-1 空位、-2 分页、页签数=页数否则崩溃；[NPC] 索引与 itemshop.lst 注册码两套编号辨析；价格取物品 [price]；每日/副本/人偶变体商店；商城与分解商店系统辨析）
- 更新：index（37 页）
- 备注：草案先经用户确认（2026-11-05）再入库，符合"用户确认门槛"流程；来源=pvfDoc【06】6 篇教程 + 封包实测 equipmentshop7.shp 与 itemshop.lst（~170 商店）

## [2026-11-05] update | 乘数判别完成——班图金币探针（用户实测）
- 更新：concepts/npc-price（溢出节改判"完成"：乘数 ×200÷1000 三实测点确认；安全上限提升 [value]≤10,737,418 / 到手≤2,147,483；新增班图族金币参照物：price 500,000 / value 2,500,000 / 卖价恰等于买入价）、debugging/pink-price-incident（后续二：×1000 淘汰，未决问题闭环；现有 2,000,000 固定价仍在安全域无需返工）、index 与 AGENTS.md 的 npc-price 行同步
- 判别依据：用户实测班图金币（3340）游戏内卖价 500,000 = 2.5M÷5 无溢出；×1000 模型预测 −358,993 被淘汰
- 待办：entities/design-recipe（设计图，含回购型节）草案已呈用户，待确认后入库

## [2026-11-05] ingest | 设计图知识（用户确认后入库）
- 新增：entities/design-recipe（recipe 类 .stk：文件模板、[int data] 配方结构【N×材料 + M×产出，实测2件程序化验证】、三种工艺类型、最低制作等级部位表、掉落控制追加段；回购类型设计图一节【装备→3340 班图金币变现，用户告知，含制作模板与"封包内无现成实例"实况】；10424 对 [price] 的"回收价格"注释与 npc-price 记矛盾待验证）
- 更新：index（38 页）、AGENTS.md 目录详情同步
- 备注：草案两轮呈报用户（首版 + 回购节增补）后确认入库；tags 含"回购"保证回购物品问题可检索命中

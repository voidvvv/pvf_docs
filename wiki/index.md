---
title: 知识库目录（index）
category: meta
tags: [目录, 导航]
sources: []
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 知识库目录（index）

> 共 37 页（4 根级 + 9 概念 + 11 实体 + 4 流程 + 3 排错 + 6 来源）· 最后更新 2026-11-05
> 维护协议见 [[WIKI]] · 全局知识地图见 [[overview]] · 操作历史见 [[log]]

## 根级（meta）

- [[WIKI]] — Schema：任何 LLM 接手维护前必读（目录约定 / 页面格式 / Ingest·Query·Update·Lint 协议）
- [[overview]] — 总览：知识地图 + 五条主线 + 当前知识缺口
- [[index]] — 本目录
- [[log]] — append-only 操作日志

## concepts/（概念）

- [[huanjing-path-mapping]] — huanjing/ 前缀 ⇄ DNF 内路径映射，MCP 传参不带前缀
- [[pvf-format]] — PVF 封包与文件类型体系（.etc/.nut/.skl/.apd/.obj/.lst…）及 LST 索引机制
- [[etc-table-format]] — .etc 数据表语法（[section] 分段、Tab 分隔）与修改铁律
- [[probability-basis]] — 概率基数 1,000,000 约定、稀有度累计阈值差值算法
- [[nut-scripting]] — nut/Squirrel 脚本体系：目录结构、命名规则、钩子、模组命名空间、静默异常特性
- [[load-state]] — load_state 注册机制：pushScriptFiles / pushState / pushPassiveObj 与加载顺序
- [[appendage]] — Appendage 两种定义方式、CNSquirrelAppendage API 全表、钩子、陷阱
- [[passive-object]] — 被动物体三件套（.obj + po_*.nut + lst）、技能→po 数据传递
- [[npc-price]] — [price]=NPC买入价 / [value]=卖出基础价÷5；溢出上限 [value]≤10,737,418（乘数×200 已三实测点判别）

## entities/（实体）

- [[worlddrop-etc]] — etc/worlddrop.etc 世界掉落：等级块结构、-1 分隔、独立判定、修改要点
- [[itemdropinfo-family]] — 掉率三件套（hell/monseter/extra）：段落详解、实测爆率、倍率系统
- [[equ-format]] — .equ 装备定义格式：字段表、rarity 映射（3=粉，文档双源）、未证实字段清单
- [[equipment-lst]] — equipment.lst 两行记录格式（代码行+反引号路径行）、勿用 MCP 整读
- [[npc-shop]] — NPC 商店 itemshop/.shp：sell item 编码（-1空位/-2分页/页签对应）、两套编号辨析、价格取物品[price]
- [[quickbar-equipment-system]] — 快捷栏装备三函数协作机制（itemIndex 共享向量）
- [[crest-items]] — 勇者纹章道具族 100000101~110 对照
- [[pvfutility-mcp]] — 工具：pvfutility MCP 接口速查与标准修改流程
- [[frida-js]] — 工具：frida.js 运行时 hook（与 PVF 修改互补）
- [[pvfdoc-library]] — 参考库导航：pvfDoc 19 类教程目录与按需 ingest 约定
- [[input-raw-references]] — 参考资料索引：input/raw 五个 txt（钩子全表/职业枚举/po 结论）

## howto/（流程）

- [[modify-drop-rate]] — 修改怪物/世界掉率完整流程 + 五类需求配方
- [[add-quickbar-equipment]] — 新增一件快捷栏装备（纹章）四步流程与代码模板
- [[modify-nut-script]] — 修改/新增 .nut 脚本通用流程与检查单
- [[analyze-skill]] — 用 dnf-skill-analyzer 定位/读取/解析 .skl 三步法

## debugging/（排错）

- [[procappend-priest-bug]] — 纹章失效案例：静默异常阻断每帧函数（根因实测确认）+ 排错方法论
- [[naming-traps]] — 陷阱集：原版拼写（monseter）等命名红线
- [[pink-price-incident]] — 粉装售价事故：int32 溢出负价（-89,934 数值链）+ 导出副本≠封包实况导致遗漏

## sources/（来源摘要）

- [[src-drop-rate-doc]] — 物品掉率文件修改说明（掉率系统母文档）
- [[src-agents-md]] — AGENTS.md 项目说明书（约定与工具母文档）
- [[src-quickbar-guide]] — 快捷栏装备系统与 Appendage 使用指南
- [[src-my-inclusion]] — input/my_inclusion.md（nut/PVF 入门）
- [[src-gift-box]] — gift_box_analysis.md（礼盒系统，仅摘要未拆页）
- [[src-atswordman-render]] — atswordman_render_analysis.md（渲染链，仅摘要未拆页）

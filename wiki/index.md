---
title: 知识库目录（index）
category: meta
tags: [目录, 导航]
sources: []
created: 2026-11-04
updated: 2026-11-06
status: current
---

# 知识库目录（index）

> 共 49 页（4 根级 + 10 概念 + 19 实体 + 7 流程 + 3 排错 + 6 来源）· 最后更新 2026-11-06
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
- [[dungeon-chain]] — 地下城→房间→怪物三层引用链、.map [monster] 行 10 字段解码、lst 格式家族（两行 ID 在前 / stackable 单行）、全系统仅两处真随机

## entities/（实体）

- [[worlddrop-etc]] — etc/worlddrop.etc 世界掉落：等级块结构、-1 分隔、独立判定、修改要点
- [[itemdropinfo-family]] — 掉率三件套（hell/monseter/extra）：段落详解、实测爆率、倍率系统
- [[equ-format]] — .equ 装备定义格式：字段表、rarity 映射（3=粉，文档双源）、未证实字段清单
- [[equipment-lst]] — equipment.lst 两行记录格式（代码行+反引号路径行）、勿用 MCP 整读；⚠️ stackable.lst 封包实况为单行格式，改 lst 先 GET 实况
- [[npc-shop]] — NPC 商店 itemshop/.shp：sell item 编码（-1空位/-2分页/页签对应）、两套编号辨析、价格取物品[price]；单页可载 900+ 件、赛丽亚已扩 13 页签（含 8 回购页签）
- [[design-recipe]] — 设计图（recipe 类 .stk）：int data 配方结构（M≥2 无先例勿用）、三种工艺、掉落控制；回购系统已全量落地（3,344 张回购图+回购金币 796000000，等级×10 万变现）
- [[monster-dir]] — monster/ 目录与 .mob：字段表、新旧双体系、数值三层模型（基础表×倍率×难度表）、lst 注册与孤儿
- [[monster-act]] — .act 行为语言：MOTION/TRIGGER/BEHAVIOR 骨架、WHICH 对象筛选、DIM 变量与触发器开关=阶段机、[ME] n 全文件 0 基寻址
- [[monster-behavior-commands]] — BEHAVIOR 指令字典（位移/判定/召唤/状态/演出五类速查+用例配方）
- [[monster-ai]] — [ai pattern] 四件套职责、.ai 条件树语法、[attack kind] 12 列解码、驱动模型
- [[monster-atk]] — .atk 命中判定字段表与三表对齐规则
- [[dungeon-dir]] — .dgn 字段表、[greed] 连通图字母表、多套 [maze info]、塔类/疲劳/深渊开关、etc 桥表、sqr/map 钩子
- [[map-dir]] — .map 房间字段、怪物摆放、.til 碰撞网格、深渊房两形态、[dummy] 假 boss 房
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
- [[add-monster]] — 新增一只怪物：选模板→.mob/三件套/AI→注册 lst→摆进房间→导入打包 + 检查单
- [[modify-monster]] — **改怪物机制总入口：需求→配方速查表**（数值/行为/打感三类，说出机制即查即改）
- [[modify-dungeon-map]] — 改地下城与房间：排怪/加房/等级/免疲劳/深渊/塔层 需求→落点

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

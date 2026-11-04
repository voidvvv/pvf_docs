---
title: 参考库导航：pvfDoc 19 类教程
category: entity
tags: [pvfdoc, 教程, 参考]
sources: ["AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 参考库导航：pvfDoc 19 类教程

`pvfDoc/pvfDoc/`（源工作区 `D:\pvf` 内）是按主题分类的 PVF 修改教程库（JSON 格式）。**本页只做导航索引；按需 ingest——遇到对应主题问题时，读取该类教程并按 [[WIKI]] §5.1 把结论沉淀为 wiki 页。**

## 分类目录

| 目录 | 主题 | 何时查阅 |
|------|------|----------|
| 【01】PVF文件解读 | PVF 格式基础 | 疑问涉及 [[pvf-format]] 未覆盖的格式细节 |
| 【02】装备修改 | 装备（.equ） | 新增 / 修改装备属性 |
| 【03】怪物修改 | 怪物 | 怪物 AI / 属性 |
| 【04】技能修改 | 技能（含中英对照、全职业对照表） | [[analyze-skill]] 未覆盖的技能改法 |
| 【05】APC修改 | APC | APC 相关 |
| 【06】NPC-商店 | NPC / 商店 | 商店贩卖列表 |
| 【07】任务修改 | 任务 | 任务奖励 / 条件 |
| 【08】地图城镇修改 | 地图 / 城镇 | 地图配置 |
| 【09】时装修改 | 时装 | 时装外观 / 属性 |
| 【10】物品修改 | 物品（.stk 等） | 新道具（参考 [[src-gift-box]]） |
| 【11】其他PVF杂项 | 杂项 | 兜底 |
| 【12】高级教程 | 高级 | 复杂机制 |
| 【13】obj | 被动物体 | [[passive-object]] 深入 |
| 【14】宠物修改 | 宠物 | 宠物系统 |
| 【16】常用模板【复制可用】 | 可复制模板 | **动手写新内容前先来找模板** |
| 【17】小白玖仙的自学模式 | 入门 | 新手路线 |
| 【18】掉落 | 掉落系统 | [[itemdropinfo-family]] / [[worlddrop-etc]] 未覆盖的掉落机制 |
| 【19】活动 | 活动 | 活动配置 |

（另有【15】空缺，为原库编号跳号。）

## 使用约定

- 教程为 JSON 格式，用 Read / `get_file_content` 读取；
- wiki 页面引用某教程结论时，在 frontmatter `sources` 写相对路径（如 `pvfDoc/pvfDoc/【18】掉落/xxx.json`）；
- 教程与 wiki 已有页冲突时按 [[WIKI]] §5.3 记矛盾，**以实测为准**。

## 相关

- 另一参考资料库：[[input-raw-references]]
- 来源摘要：[[src-agents-md]]

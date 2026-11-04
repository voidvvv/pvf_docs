---
title: 总览：知识地图与综合结论
category: meta
tags: [总览, 导航]
sources: ["AGENTS.md", "物品掉率文件修改说明.md", "快捷栏装备系统与Appendage使用指南.md", "input/my_inclusion.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 总览：知识地图与综合结论

> 全库导航的"第二入口"（第一入口是 [[index]] 的分类目录）。本页回答"这个知识库整体讲什么、各部分怎么串起来"。

## 一句话画像

本库沉淀 **DNF 86幻境版本 PVF 模组开发** 的全部既有知识：数据表（.etc）与脚本（.nut）的结构解读、修改方法、排错经验、工具用法。数据本身不进库——用 [[pvfutility-mcp]] 实时查。

## 知识地图

```
                    PVF 封包（Script.pvf）
                          │
        ┌─────────────────┼──────────────────┐
     数据表 .etc         脚本 .nut            其他资源
        │                  │                .skl/.apd/.obj/.stk/.chr…
 ┌──────┴──────┐    ┌──────┴──────┐               │
 掉率系统     其他表  注册机制    运行时体系          渲染链（待扩）
 │                    │               │
 ├ 三件套             ├ load-state    ├ appendage（状态）
 │ [[itemdropinfo…]]  │ [[load-state]]│ [[appendage]]
 ├ 世界掉落           │               ├ 被动物 [[passive-object]]
 │ [[worlddrop-etc]]  │               └ 快捷栏装备应用
 └ 概率算法           └ 语法体系        [[quickbar-equipment-system]]
   [[probability-basis]] [[nut-scripting]]
```

## 贯穿全库的五条主线

1. **一切修改的终点是重新打包**：任何 `.etc` / `.nut` 改动 → `import_file` → `save_as_pvf` → 替换游戏 PVF（[[pvfutility-mcp]]）。
2. **概率只有一种基数**：1,000,000 = 100%，累计阈值差值法判定稀有度（[[probability-basis]]）。
3. **脚本靠注册与钩子**：load_state 三原语注册（[[load-state]]），引擎在固定时机回调钩子（[[nut-scripting]]）。
4. **状态效果皆 appendage**：两种定义方式一套 API（[[appendage]]），快捷栏装备是它的代表性应用（[[quickbar-equipment-system]]）。
5. **静默失败是头号陷阱**：Squirrel 异常被引擎吞掉 → 函数后半段不执行且无报错（[[procappend-priest-bug]]）；数据侧的对应陷阱是改名 / 错位（[[naming-traps]]）。

## 当前综合结论（bootstrap 时点）

- 掉率系统已完整摸清：三件套实测爆率 + worlddrop 全结构，可支撑任意掉率改造（[[modify-drop-rate]]）。
- 脚本体系主干已通：注册 → 钩子 → ap / po，快捷栏装备可作为新装备类效果的模板。
- **知识缺口（下次探索方向）**：
  - 礼盒系统（[[src-gift-box]]）与渲染链（[[src-atswordman-render]]）只有摘要，未拆页；
  - `.atk` 攻击信息、地图 / 怪物脚本领域仍空白（`.equ` 格式与 rarity 映射已于 2026-11-04 沉淀为 [[equ-format]]）；
  - rarity=6/7（文档未记载档位，实测共 1,141 件）含义待证实；`[drop prob]` 第 4/6/7 列的掉落类别待客户端确认；
  - `pvfDoc/` 19 类教程未消化（按需，见 [[pvfdoc-library]]）。

## 给跨会话 AI 的最短路径

拿到本库后：读本页了解全貌 → 需要细节时查 [[index]] → 按协议（[[WIKI]]）回答 / 维护。

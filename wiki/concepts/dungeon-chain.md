---
title: 概念：地下城→房间→怪物三层引用链
category: concept
tags: [地下城, 房间, 怪物, 引用链, lst]
sources: ["pvfDoc/pvfDoc/【08】地图城镇修改/副本/11407_dungeon文件夹.json",
         "pvfDoc/pvfDoc/【08】地图城镇修改/副本/11448_地图副本dng以及map详解.json",
         "pvfDoc/pvfDoc/【08】地图城镇修改/副本/11764_副本接口wdm.副本dgn,小房间map解读.json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 概念：地下城→房间→怪物三层引用链

怪物怎么出现在地下城里？**三层全是"逐只/逐格钉死"的静态数据**，由 lst 索引串联：

```
worldmap/*.wdm                 区域聚合（[dungeon] ID 列表）
   ↓ dungeonID
dungeon/dungeon.lst ──→ <包>/<名>.dgn          布局/难度/疲劳/掉落（[[dungeon-dir]]）
   ├ [size] 网格 + [greed] 连通图（每房横向占 2 字符）
   ├ [map specification] x y mapID…  房间钉死网格（多 ID=随机抽）
   ├ [start map] / [boss map specification]
   └ [seal door map index]  深渊房
   ↓ mapID
map/map.lst ──→ <数字_图名>/(x,y)MapID_角色.map   单个房间（[[map-dir]]）
   └ [monster] 每行钉死一只怪
   ↓ monsterID
monster/monster.lst ──→ <家族>/<名>.mob          怪物本体（[[monster-dir]]）
```

- `.map` 内 `[dungeon]` 段是**反向归属锁**：跨地下城引用会 bug/闪退（=doc 11764 + 实证）；同名房间跨本复用必须物理复制 + 新 ID。
- 查某本全部房间：**用 `[dungeon] 段正则搜**，别按目录数文件（~838 个 .map 是未注册孤儿）。
- 文件名里的坐标可能与 dgn 实际钉位不符（gold_cave 8114/8117 写 (3,3) 实用 8116）——**以 dgn [map specification] 为准**。

## .map [monster] 行 10 字段解码（证据最强版）

实例：`66417 1 0 948 317 0 2 1 [fixed] [boss]`

| 位 | 含义 | 证据 |
|---|---|---|
| 1 | 怪物 ID → monster.lst | 实证（三方一致） |
| 2-3 | **等级规则**：`1 0`=吃 dgn `[basis level]`；`1 N`=基础+N；`0 N`=无视 dgn 锁死 N 级 | =doc 11764 逐例 + 金矿洞实测 |
| 4-6 | 房内 x y z 坐标 | 实证 |
| 7 | 出场标识（观测：普通怪=1、boss 行=2） | 观测（教程"可能是出场方式"） |
| 8 | 恒 1，未知 | 全库普查 |
| 9 | `[fixed]`（全库 28,048 处唯一取值） | 全量普查 |
| 10 | `[normal]`/`[boss]`/`[champion] N`（N=异常怪强度 0~100） | =doc 11764 + 实测 |

## lst 格式家族（改任何 lst 前先确认格式）

| lst | 格式 | 实例 |
|---|---|---|
| equipment / monster / dungeon / map | **两行记录：ID 行在前，路径行在后**（路径相对本目录，不带目录前缀） | `1\r\n\`goblin/goblin.mob\`` |
| stackable | **单行**：`ID\t\`路径\`` | `3340\t\`material/gold_coin.stk\`` |
| passiveobject | 单行同上 | `10687\t\`ActionObject/…\`` |

⚠️ 行序曾有两说，2026-11-05 用已知配对（66417=widire_q）裁决为 **ID 在前**。ID 全部稀疏（深渊 5xxxx/6xxxx、活动 20000+、甚至日期式 ID `2023120808`），**按行号推 ID 必错**。

## 全系统仅有的两处真随机

1. 房间在迷宫槽位间落位（`[greed]`）+ `[map specification]` 多 ID 抽取；
2. 深渊小队（`etc/hellparty.etc`）。

其余（怪物是谁、几只、站哪、几级）全部静态钉死。

## 相关

- 各层详解：[[dungeon-dir]] · [[map-dir]] · [[monster-dir]] · [[monster-act]]
- 操作流程：[[add-monster]] · [[modify-monster]] · [[modify-dungeon-map]]
- lst 工具坑：[[pvfutility-mcp]]

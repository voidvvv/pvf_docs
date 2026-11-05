---
title: 实体：dungeon/ 地下城定义与 .dgn
category: entity
tags: [dungeon, dgn, 地下城, greed]
sources: ["pvfDoc/pvfDoc/【08】地图城镇修改/副本/11448_地图副本dng以及map详解.json",
         "pvfDoc/pvfDoc/【08】地图城镇修改/副本/11502_副本map方向.json",
         "pvfDoc/pvfDoc/【08】地图城镇修改/副本/11503_副本各种代码.json",
         "pvfDoc/pvfDoc/【08】地图城镇修改/PVF修改思路-地图篇/11407_dungeon文件夹.json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 实体：dungeon/ 地下城定义与 .dgn

`dungeon/`（918 文件：.dgn 472 / .ani 375 进场标题 / .tbl 48 / .lst 2）。`dungeon.lst` 两行记录（ID 在前，~455 条，ID 稀疏）。官方空白模板：`dungeon/dungeonsample.dgn`（**做新本从它起步**）。

## 目录组织

按内容包分目录（act1~9 主线 / ancient 远古 / anton 安图恩 / towers 塔 / timegate 时空门 / riskdungeon 异界(R_ 系) / stormofmetastasis 大转移 / villageattack 攻城 / longzhilu 活动层…），每包含 `title/`（进场标题 ani）与 `table/`（本包难度 tbl）。

## .dgn 关键字段

| 组 | 字段 | 说明 |
|---|---|---|
| 展示 | `[name]/[explain]`（中文内联）、`[entering title]`、`[cutscene image]`、`[minimap image]`、`[worldmap pattern info]`（区域号+槽位） | 入口层 |
| 等级难度 | `[minimum required level]`、**`[basis level]`（副本基础等级，怪物 `1 0` 规则吃它）**、`[recommended level]`、`[experience increasing point]`、`[designate dungeon difficulty]`、`[required item]`（门票）、`[coin limit]`、`[limit party count]/[necessary party]/[limit inout count]`、`[prohibit practice]` | =doc 11503（旧文 11448 两义互换，以 11503 为准） |
| 疲劳 | **无 etc 表——按房间扣点，`[no fatigue]` 空标记=豁免**（114 处，活动本标配） | 实证 |
| 开关 | `[ancient dungeon]/[hell dungeon](110=可出深渊)/[risk dungeon]/[escape hell]`、`[dungeon type]`（tower of death/advance altar…） | |
| **布局** | `[size]` 网格宽高；`[greed]` 连通图；`[maze info]`（**一套布局的起始标记，可并列多套**——任务前后/多走线，只读第一套会漏改）；`[map specification] x y mapID…`（房间→地图池，多 ID 随机抽，可省略=按 .map 的 [dungeon]+[type] 随机）；`[start map]/[boss map (specification)]`；`[pathgate object]` 10 槽门 obj；`[seal door map index]/[seal door pos]/[hit count]/[seal door appear rate]` 深渊房 | =doc 11407/11503 |
| 刷怪掉落 | `[special passive object item]`（可碎物掉落分组）、`[event monster]`、`[clear condition]`、`[monsterapc diff table]`（换本包难度表） | |
| 塔类 | 无网格：`[death tower map indexes]`（首数=总层数，层+mapID 对）、`[tower dialog]`（每层台词）、`[tower recovery]/[revision table]` 等 | |

## [greed] 连通图（=doc 11502）

每行字符串、**每房间横向占 2 字符**（不看 [size] 直接数字符会房间数翻倍）；字母=开门方向：

`A`不通 `B`右 `C`上 `D`右上 `E`左 `F`左右 `G`左上 `H`左上右 `I`下 `J`右下 `K`上下 `L`上下右 `M`左下 `N`左下右 `P`十字

## etc/ 桥表（本地无 etc/，**走 MCP 读封包**）

| 表 | 作用 |
|---|---|
| `worldmap/<区>.wdm` | 区域→地下城 ID 列表 + `[hell dungeon]` 深渊开启任务 + `[hell freepass item]` 通行证 |
| `etc/itemdictionary/(r)dungeon_name.etc` | 词典"获取地点"：flag=1 行把多个深渊变体聚合到同槽 |
| `etc/itemdictionary/(r)dungeondropinfo.etc` | 副本→掉落等级段（普通/深渊两套，按 5 级递增） |
| `etc/hellparty.etc` | 深渊小队构成（[group] 内 A/B=非常困难/困难档） |
| `etc/deathtower.etc` 等 | 塔/终极本奖励参数 |

## sqr/map 钩子

`.dgn/.map` 管静态；`sqr/map/*.nut` 管运行时：`sq_GetDungeonByStage()→sq_GetDuegonIndex()`（**Duegon 是 API 原文 typo，勿纠正**）+`sq_GetMapIndex()` 命中 (dungeonID,mapID) 即挂 appendage（anton.nut 实证）。分工同 [[appendage]] 体系。

## 坑

大小写双标（lst 写 `Act1/Lorien.dgn` 磁盘是 `act1/`）；tbl 文件名 typo（kingsruins**ficulty**bonus.tbl 勿改）；[champion]/[hit count] 口径/[schedule] 69 未解。

## 相关

[[dungeon-chain]] · [[map-dir]] · [[modify-dungeon-map]] · [[pvfutility-mcp]]

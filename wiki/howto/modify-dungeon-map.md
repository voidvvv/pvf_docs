---
title: 流程：修改地下城与房间
category: howto
tags: [howto, 地下城, 房间, 排怪]
sources: []
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 流程：修改地下城与房间

> 背景知识：[[dungeon-dir]]（.dgn）、[[map-dir]]（.map）、[[dungeon-chain]]（引用链）。查某本全部房间用 `[dungeon] 段搜，别按目录数文件。

## 需求→落点

| 需求 | 落点 |
|---|---|
| 房间换怪/加减怪 | `.map [monster]` 行：改 ID/等级规则/坐标；加行即加怪（每行一只，字段表 [[dungeon-chain]]） |
| 整房替换 | dgn `[map specification] x y 新mapID`（多 ID=随机池） |
| 加房间 | ①复制近似 .map 改（[dungeon] 段必须=本本 ID）②map.lst 注册新 ID（两行）③dgn 加 [map specification] 钉位 ④[greed] 连通图与 [size] 同步扩 |
| 挪起点/Boss 房 | dgn `[start map]`/`[boss map (specification)]` 坐标对 |
| 副本等级 | dgn `[basis level]`（怪物 `1 0` 规则跟随）+ `[minimum required level]` |
| 免疲劳 | dgn 加 `[no fatigue]` 空标记（疲劳=按房间扣点，无表） |
| 可出深渊 | dgn `[hell dungeon]` 110 + `[seal door map index/pos]` 指深渊房 + 深渊房 .map 钉 `[hellparty]`（小队在 etc/hellparty.etc） |
| 可碎物掉落 | .map `[special passive object]` `[item]` 参数首数 → dgn `[special passive object item]` 分组号 |
| 改通行区域 | 房间 tile/ 的 .til `[pass type]` 数字网格（14 列×N 行，2/0=通/阻） |
| 塔加层 | dgn `[death tower map indexes]`（首数=总层数，层+mapID 对）+ `[tower dialog]` 台词 |
| 新地下城 | 复制 `dungeon/dungeonsample.dgn` 起步 → dungeon.lst 注册 → 房间模板 `map/mapsample.map` → worldmap/*.wdm 挂区域槽位 |

## 操作要点

1. **先读封包实况**（MCP GET）再编辑——本地副本可能过期（[[pvfutility-mcp]] 教训）；
2. 改 .map 数量/位置后，dgn 的 [greed]（每房横向占 2 字符！）与 [size] 必须一致，多套 `[maze info]` 布局**每套都要改**（漏改=某走线错乱）；
3. 新 .map 的 `[dungeon]` 段、lst 条目、dgn 引用三处 ID 互证后才算闭环；
4. 页签/`[tab name]` 类规则不适用于此（那是 NPC 商店，见 [[npc-shop]]）；
5. 完成后 save_as_pvf + 换文件；游戏内开本实测：每个门能开、boss 房能通关、[dummy] 房确认无误用。

## 排错入口

清完 boss 门不开 → 查 .map `[type]` 是否 `[dummy]`；进图闪退 → [dungeon] 段跨本引用/地图 ID 未注册；房间数翻倍 → greed 换算忘了每房 2 字符。

## 相关

[[modify-monster]]（改怪）· [[add-monster]]（新增怪）· [[dungeon-chain]]

---
title: 实体：map/ 房间定义与 .map
category: entity
tags: [map, 房间, til, 怪物摆放]
sources: ["pvfDoc/pvfDoc/【08】地图城镇修改/副本/11764_副本接口wdm.副本dgn,小房间map解读.json",
         "pvfDoc/pvfDoc/【08】地图城镇修改/副本/11448_地图副本dng以及map详解.json"]
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 实体：map/ 房间定义与 .map

`map/`（36,211 文件：.map 4,783 / .til 9,365 / .ani 21,595）。`map.lst` 两行记录（ID 在前，3,945 条；~838 个 .map 未注册=孤儿）。官方模板：`map/mapsample.map`。

## 目录与命名

- 子目录 `数字_地图名`（144~225，数字=地下城 ID，与 .dgn/[dungeon] 段互证）或无前缀老目录（Lorien…）；
- 文件名 `(x,y)MapID_角色.map`：s=start、n/纯数字=normal、b/bn=boss、named=精英、hell_=深渊、q=任务房（部分推测）。

## .map 关键 section

| section | 说明 |
|---|---|
| **`[dungeon]`** | 归属地下城 ID（**反向锁**，跨本引用闪退） |
| `[type]` | `[normal]`/`[boss]`/`[dummy]`（33 间假 boss 房：杀了不开门） |
| `[greed]` | 本房开门方向字母（同 [[dungeon-dir]] 字母表） |
| `[tile]/[extended tile]` | 地板（相对本目录 tile/） |
| `[pathgate pos]` | 4 组 (x,y)=左右上下门坐标 |
| `[dungeon start area]` | 玩家出生点 |
| `[background animation]/[animation]` | 背景/装饰 ani（同名 .als 挂特效） |
| **`[monster]`** | **逐只钉怪**——10 字段解码见 [[dungeon-chain]]（ID/等级规则/xyz/类型） |
| `[monster specific AI]` | 逐怪 AI 覆盖 |
| `[special passive object]` | 两行一组：`objID x y z 数量` + `[item]`（可碎物掉落，首数=dgn 分组号）/`[trap]`/`[monster]`（爆怪） |
| `[NPC]` | 房内 NPC |
| `[far/middle/near sight scroll]` | 视差速度 |
| `[sound]` | BGM(M_)+环境音(AMB_) |
| `[champion]/[fix champion]` | 房间级异常怪 |

## .til 结构

- `[IMAGE]`：指向**客户端** .img（贴图不在 PVF 内）；
- `[pass type]`：14 列×N 行数字网格=**可行走碰撞图**（2=可走/0=阻挡，推测）——**改通行区域就改这个网格**；
- 命名：`tileXX_<mapID>.til`=某房专用变体、`*_ex.til`=扩展层。

## 深渊房两种形态

1. **真深渊房**（如 `grakkarak/hell_97.map`）：结构同普通房 + `[special passive object]` 钉深渊柱（30519）+ `[hellparty]` 波次块（每行：小队/怪ID、20/25、波次）；小队构成在 `etc/hellparty.etc`；
2. **普通房深渊变体**：同一房间注册为新 ID（3908→3913 式）；
3. ⚠️ `xxx.map(hell)` 后缀文件（3 个）是**纯孤儿备份残留**，游戏不读取。

## 坑

文件名坐标≠dgn 实际钉位（以 [map specification] 为准）；[map name] 几乎全是占位"PVP 无名"；.str 是 CP949/CP932 勿强改；改通行区域=改 .til [pass type]。

## 相关

[[dungeon-chain]] · [[dungeon-dir]] · [[monster-dir]] · [[modify-dungeon-map]]

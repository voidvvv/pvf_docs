---
title: 设计图（recipe 类道具）
category: entity
tags: [设计图, recipe, stackable, 制作, 回购]
sources: ["pvfDoc/pvfDoc/【10】物品修改/10415_设计图解读.json",
         "pvfDoc/pvfDoc/【10】物品修改/道具材料分类/10424_设计图.json",
         "pvfDoc/pvfDoc/【10】物品修改/10403_勇者装备改造.json"]
created: 2026-11-05
updated: 2026-11-05
status: current
---

# 设计图（recipe 类道具）

设计图是 `.stk` 可堆叠道具，`[stackable type]` = **`[recipe]`**（附参数 1）。使用时消耗 `[int data]` 列出的材料，产出指定装备/道具。实测位于 `stackable/recipe/` 目录（约 **2,403 张**，2026-11-05），可被 NPC 出售（如罗杰·莱文的 itemproduct 商店，见 [[npc-shop]]）。

## 文件模板（10415 + 10424 合并注释）

```
#PVF_File
[name]              `XX设计图`
[grade]             90            // 等级组
[rarity]            4             // 品阶
[usable job]        `[all]`       // 可用职业
[attach type]       `[free]`      // 交易类型
[minimum level]     90            // 使用等级
[weight]            10
[need material]     20181215  10  // 10415 注释为"购买条件"
[icon]              `Item/stackable/recipe.img`  35
[field image]       `Item/FieldImage.img`  28    // 地图掉落图标
[stackable type]    `[recipe]`  1 // 道具类型=图纸
[move wav]          `PAPER_TOUCH`
[int data]          …（见下，配方核心）
[string data]       `[weaving]`   // 工艺类型（见下）
[success rate]      100           // 成功率（实测样本可缺省，缺省含义未验证）
[maintain ability]  1             // 继承强化/增幅/锻造/宝珠
[broadcast type]    1             // 制作成功上公告
[fail output]       102020300  1  // 失败补偿物（代码 数量）
[use upgrade table]               // 保留增幅锻造升级
[use 2nd upgrade table]           // 使用第二增幅表
```

## [int data] 配方结构（核心，实测验证通过）

```
[int data]
  5                      ← ① 材料组数 N
  80000002   1           ← ② N 组「材料代码 数量」
  20181215   200
  3823       100
  20771104   2000
  1                      ← ③ 产出种类数 M
  79000002   1   0       ← ④ M 组「产出代码 数量 未知(恒0)」
[/int data]
```

实测样例（2026-11-05，结构解析 ✅）：

- `stackable/recipe/2651326.stk`（伪装者的迷惑之戒设计图）：材料 10000205×150、10000224×5、3167×750、3263×50 → 产出 100320086×1；
- `stackable/recipe/nrcp_ring_2659531.stk`（魔法封印的红月银戒指设计图）：材料 3157×3、3053×1、3161×6、3166×1 → 产出 24217×1。

## [string data] 工艺类型（三选一）

| 值 | 含义（10424） |
|----|----|
| `[weaving]` | 编织 |
| `[craftmanship]` | 工艺（制作：炼金、人偶） |
| `[machinary]` | 机械 |

三者都可制作装备，具体对应关系教程原文称"未知"。

## [minimum itemmaking level] 最低制作等级

两列：`部位` `等级`，如 `harmor 5`。部位参数：`weapon` 武器、`wrist` 手镯、`amulet` 项链、`ring` 戒指、`cloth` 布甲、`harmor` 重甲、`larmor` 轻甲、`leather` 皮甲、`plate` 板甲。

## 掉落控制追加字段（10424「真·赤炼肩甲」实例）

| 字段 | 含义 / 取值 |
|------|------|
| `[special monster drop]` 1 | 独立怪物掉落开关（默认 1） |
| `[difficulty for extra drop]` | 难度额外掉落：`easy`/`medium`/`hard`/`ultimate`/`hero` |
| `[dungeon type for extra drop]` | 地下城类型额外掉落（如 `[all]`） |

## 回购类型设计图（装备变现模式）🏷回购

一类特殊用法的设计图（用户告知，2026-11-05）：**以装备为原料、以金币道具为产出**——把装备"转换"成可卖 NPC 的金币道具，实现装备变现。

### ✅ 已全量落地（2026-11-05，用户确认方案后实施）

**60 级以上全部粉装（rarity=3）回购系统**已写入封包：

| 要素 | 值 |
|------|-----|
| 回购图数量 | **3,344 张**（有 equipment.lst 代码者；另有 284 件未注册 LST 无法引用而排除，清单在源工作区 `demo/rebuy_excluded_no_code.csv`） |
| 代码段 | **796000000**（回购金币）+ **796000001~796003344**（回购图），写入前已验空 |
| 文件位置 | `stackable/pink_rebuy/`（本地副本 `huanjing/stackable/pink_rebuy/`） |
| 单图产出 | `[int data]`：材料=装备×1 → 产出=**回购金币 × 等级数**（如 60 级=60 枚=600 万） |
| 回购金币 | `796000000.stk`：`[price]` 100,000、`[value]` 500,000 → **卖店恰 100,000**（10 万面值，与 3340 同构自洽） |
| NPC 上架 | 赛丽亚 `equipmentshop7.shp` 新增 8 个等级页签：回购60/65/70/75/80/85/86/90+（68→65 签、71→70 签归档；最大页 765 件，低于辛达商店 922 件/页实证容量） |
| 图纸售价 | `[price]`=1,000（近似免费领） |

模板（镜像实测样本 `stackable/recipe/2651326.stk`，去掉 `[minimum itemmaking level]` 以免制作等级门槛）：

```
[int data]
  1                    ← 材料组数
  <装备代码>  1        ← 被回购装备（equipment.lst 代码）
  1                    ← 产出种类数（勿用 ≥2，见下）
  796000000  <等级> 0  ← 产出：回购金币 × 等级
[/int data]
```

使用流程：赛丽亚商店对应等级页签领回购图（1,000 金）→ 持装备使用（weaving 制作，无制作等级要求）→ 得等级×10 万金币。

### ⚠️ 结构红线（2026-11-05 全量扫描 2,432 张既有设计图得出）

- **产出种类数 M≥2 全封包零先例**（全部 M=1）——需要多种产出时不要直接写 M=2，应实测验证或改用单产物组合；
- **装备代码作材料有 34 件先例**（如 `stackable/recipe/2651400.stk` 材料 `101000027×1`）——装备引用合法；
- 既有设计图产出数量仅见 1 与 30；产出数量无小上限迹象（booster 系统一给 342）。

### 产物：班图族的金币（3340）

`stackable/material/gold_coin.stk`：`[price]` 500,000、`[value]` 2,500,000 → **实际卖店 500,000 金币**（÷5 规则，见 [[npc-price]]；用户实测确认，卖价恰等于其买入价）。

3340 面值 50 万无法整除"等级×10 万"（68/71/86 级非整除）——落地版因此改用自建 10 万面值「回购金币」单一产物，精确满足任意等级；3340 仍是溢出标定物（判别过程见 [[npc-price]] 溢出节与 [[pink-price-incident]] 后续二）。

## ⚠️ 矛盾记录（待验证）

10424 将 `[price]` 注释为"**商店回收价格**"，与 [[npc-price]]（[price]=NPC **买入**价、卖出走 [value]）不一致——倾向按 npc-price（有实测溢出数据支撑）为准，10424 注释存疑，待实测裁决。

## 坑与注意

1. **游戏内代码获取工具无法获取设计图代码**（10415 明示）——改图前先从 `stackable/recipe/` 或 stackable.lst 定位文件；
2. `[need material]` 的语义：10415 注释为"购买条件"而非制作材料（制作材料在 [int data]）；
3. 材料组数量 N、产出组数量 M 必须与后续数据严格对齐，错位即配方损坏。

## 相关系统

- **勇者装备改造**（不用设计图）：装备文件自身加 `[emancipate]` + `[input]`（材料→数量）+ `[output]`（成功产物→1）段（教程 10403，ID 1229/1230）；
- 设计图是 `.stk` 家族成员，通用字段（[price]/[value]、[expiration date] 等）参见 [[src-gift-box]] 与 [[npc-price]]。

## 证据

- 教程：10415（完整模板 + int data 逐段注释）、10424（参数详解 + 掉落追加段 + 传承设计图实例）、10403（勇者改造对照）
- 封包实测（2026-11-05）：繁体"設計圖"搜索命中 stackable/ 下 **2,403** 个 .stk；抽样 2 件（2651326.stk / nrcp_ring_2659531.stk）[int data] 结构程序化验证通过
- 用户告知（2026-11-05）：回购类型设计图的用途与 3340 产物；班图金币游戏内卖价 50 万（实测确认）
- 落地实施（2026-11-05）：生成脚本 `demo/_work/build_all.py`、导入 `import_rebuy.py`、全量审计 `audit_rebuy.py`（3,345 件逐件回读 0 异常，PASS）；实施参数用户三项确认（recipe 机制 / 等级×10 万 / 一次性全量）

## 相关

- 价格与卖出规则：[[npc-price]]　·　.stk 家族：[[src-gift-box]]　·　NPC 商店：[[npc-shop]]
- 教程库导航：[[pvfdoc-library]]　·　查询工具：[[pvfutility-mcp]]

# DNF PVF 礼盒道具系统完整分析

## 一、整体架构概览

礼盒（Gift Box）是 PVF 中的**消耗品类道具**，存储为 `.stk` 文件（stackable），注册在 `stackable/stackable.lst` 中。

### 核心运作流程

```
玩家右键使用礼盒 (.stk)
        ↓
读取 [stackable type] 确定礼盒类型
        ↓
根据类型执行不同逻辑：
  ├── [booster]         → 多轮独立随机，每轮抽一个，获得全部结果
  ├── [booster random]  → 单轮随机，按概率抽一个
  ├── [booster selection] → 玩家从分类列表中自选
  ├── [cera package]    → 直接获得全部物品
  ├── [usable cera package] → 使用后获得全部物品
  ├── [random upgradable legacy] → 复杂多轮可升级随机
  └── [legacy]          → 旧版随机盒
        ↓
通过物品代码(ItemCode)在对应的 .lst 中查找实际文件
  ├── equipment类型 → equipment/equipment.lst → .equ文件
  ├── stackable类型 → stackable/stackable.lst → .stk文件
  └── 特殊类型 → creature/pet.lst 等
        ↓
将物品发放到玩家背包
```

### 物品代码解析机制

礼盒中的奖励通过**物品代码(ItemCode)**引用，引擎根据物品类型在对应的LST索引中查找：

```
ItemCode → 查找对应LST文件 → 定位实际文件路径

示例：
  422310002 → equipment.lst → equipment/character/swordman/avatar/aura/chn_2023_newyear_aura_422300002.equ
  4356       → stackable.lst → stackable/quest/oppressive_destroyghost.stk
```

---

## 二、礼盒基础属性（所有礼盒共有）

### 2.1 通用字段

| 字段 | 说明 | 示例 |
|------|------|------|
| `[name]` | 道具名称 | `` `三觉·顿悟之境光环自选礼盒` `` |
| `[name2]` | 英文名称 | `` `Spring halo optional gift box` `` |
| `[explain]` | 功能说明 | `` `开启后，可以从2种新春光环装扮中，选择获得1个。` `` |
| `[flavor text]` | 风味文字（背景描述） | `` `<2011春节装扮礼包>` `` |
| `[grade]` | 品质等级 | `1` |
| `[rarity]` | 稀有度 | `2`=稀有, `3`=史诗 |
| `[icon]` | 图标资源路径 + 索引号 | `` `Item/stackable/cash_cn_2023.img`	13 `` |
| `[icon mark]` | 图标角标 | `` `Item/IconMark_cn.img`	75 `` |
| `[field image]` | 地面掉落显示图 | `` `Item/FieldImage.img`	30 `` |
| `[weight]` | 背包重量 | `1` |
| `[price]` | NPC售价 | `0` |
| `[stack limit]` | 最大堆叠数 | `1000` |
| `[cool time]` | 使用冷却(ms) | `1000` |
| `[minimum level]` | 最低使用等级 | `1` |
| `[usable job]` | 可使用职业 | `` `[all]` `` |
| `[attach type]` | 绑定类型 | `` `[trade]` ``可交易 / `` `[account]` ``帐号绑定 |
| `[move wav]` | 拾取音效 | `` `BONE_TOUCH` `` |

### 2.2 限制与消耗字段

| 字段 | 说明 | 示例 |
|------|------|------|
| `[need material]` | 开启所需材料 | `2680669	5`（需要5个代码2680669的物品） |
| `[npc gift disallowance]` | 禁止送给NPC | `1` |
| `[expiration date]` | 过期时间(Unix时间戳) | `1374721200` |
| `[random option unseal]` | 获得的装备带随机魔法封印 | `1` |
| `[suitable job]` | 推荐职业 | `` `[swordman]` `` |

---

## 三、六种礼盒类型详解

### 3.1 `[booster]` — 多轮独立随机礼盒

**机制**：多个 `[etc]` 块，每个块独立随机抽取1个物品，玩家获得**所有块的抽取结果**。

```
[stackable type]
    `[booster]`    0

[booster info]
    [etc]                          ← 第1轮随机
        1                          ← 本轮抽取数量=1
        4356    1000    10         ← 物品代码=4356, 权重=1000, 数量=10
    [/etc]
    [etc]                          ← 第2轮随机
        1
        4393    1000    10
    [/etc]
    [etc]                          ← 第3轮随机
        1
        4394    1000    10
    [/etc]
    [etc]                          ← 第4轮随机
        1
        7789    1000    1
    [/etc]
[/booster info]
```

**概率规则**：
- 每个块内的概率独立计算
- 权重（weight）越大，被抽中概率越高
- 单个物品概率 = 该物品权重 / 块内所有物品权重之和

**示例**：`LV.70达成礼盒（鬼剑士）`— 开启后获得4件奖励（每轮各抽1个）

#### 特殊：`[cera]` 子类型

当奖励为游戏货币/点券时，使用 `[cera]` 标签：

```
[booster info]
    [cera]
        1
        900    886295    2       ← 获得数量2，权重886295
        900    53178     3       ← 获得数量3，权重53178
        900    13294     4       ← 获得数量4，权重13294
        ...
    [/cera]
[/booster info]
```

**示例**：`OnTime神秘的解放之钥袋` — 数量越少权重越高（2个的权重886295 vs 100个的权重9）

#### 特殊：`[equipment]` 子类型

当奖励为装备时，使用 `[equipment]` 标签：

```
[booster info]
    [equipment]
        1
        2681090    20000    1    ← 称号装备，权重20000
        2681091    20000    1
        2681092    20000    1
        2681093    20000    1
        2681094    20000    1
    [/equipment]
[/booster info]
```

---

### 3.2 `[booster random]` — 单轮随机礼盒

**机制**：仅一个 `[etc]` 块，从中随机抽取1个物品。

```
[stackable type]
    `[booster random]`    0

[booster info]
    [etc]
        1                              ← 抽取数量=1
        2681243    4700    1           ← 普通称号A, 权重4700, 数量1
        2681244    4700    1           ← 普通称号B, 权重4700
        2681245    250     1           ← 中级称号A, 权重250
        2681246    250     1           ← 中级称号B, 权重250
        2681247    50      1           ← 高级称号A, 权重50
        2681248    50      1           ← 高级称号B, 权重50
    [/etc]
[/booster info]
```

**概率分析**（兔年幸运称号礼盒）：
| 称号 | 权重 | 概率 |
|------|------|------|
| 普通称号 (x2) | 各4700 | 各47% |
| 中级称号 (x2) | 各250 | 各2.5% |
| 高级称号 (x2) | 各50 | 各0.5% |

**与 `[booster]` 的区别**：
- `[booster]`：多个块 → 获得多件奖励
- `[booster random]`：单个块 → 只获得1件奖励

---

### 3.3 `[booster selection]` — 自选礼盒

**机制**：玩家从分类列表中选择物品，支持多标签页分类。

#### 基本结构

```
[stackable type]
    `[booster selection]`    0

[booster category num]
    tabCount    itemCountPerTab       ← 标签页数量, 每页可选数量

[booster selection num]
    totalSelectionsAllowed            ← 玩家总共可选择几件

[booster select category]            ← 第1个标签页
    tabIndex    subIndex
    [avatar]                           ← 物品类型: [avatar]/[equipment]/[etc]
        itemCode    count    param1    param2    ← avatar格式: 4列
        422310002    1       4         0
        422310003    1       4         0
        422310001    1       4         0
    [/avatar]
[/booster select category]

[booster category name]               ← 标签页名称
    `请选择职业`                       ← 第1行: 提示文字
    `新春光环装扮`                     ← 第2行: 标签页1名称
[/booster category name]
```

#### 单标签页示例：光环自选礼盒

```
[booster category num]
    1    1                            ← 1个标签页, 每页选1件

[booster selection num]
    1                                 ← 总共选1件

[booster select category]
    0    0
    [avatar]
        422310002    1    4    0
        422310003    1    4    0
        422310001    1    4    0
    [/avatar]
[/booster select category]

[booster category name]
    `请选择职业`
    `新春光环装扮`
[/booster category name]
```

#### 多标签页示例：成长武器箱子（6个职业标签页）

```
[booster category num]
    1    6                            ← 1次选择, 6个标签页可选

[booster selection num]
    1                                 ← 选1件

[booster select category]            ← 标签页0: 鬼剑士
    0    0
    [equipment]
        601000000    1
        601010000    1
        601020000    1
        601030000    1
        601040000    1
    [/equipment]
[/booster select category]

[booster select category]            ← 标签页1: 格斗家
    1    0
    [equipment]
        602000000    1
        602010000    1
        602020000    1
        602030000    1
        602040000    1
    [/equipment]
[/booster select category]

...（共6个标签页）

[booster category name]
    `请选择职业`
    `鬼剑士`
    `格斗家`
    `神枪手`
    `魔法师`
    `圣职者`
    `暗夜使者`
[/booster category name]
```

#### 物品条目格式差异

| 类型 | 格式 | 说明 |
|------|------|------|
| `[equipment]` | `itemCode count` | 装备/宠物/称号 — 2列 |
| `[avatar]` | `itemCode count param1 param2` | 装扮/光环 — 4列，额外参数可能与装扮品质/技能选择有关 |
| `[etc]` | `itemCode count` | 消耗品/材料 — 2列 |

---

### 3.4 `[cera package]` — 直开礼包

**机制**：开启后**直接获得所有物品**，无需选择或随机。

```
[stackable type]
    `[cera package]`    0

[package data]
    39697    1               ← 头发 x1
    40437    1               ← 脸部 x1
    40925    1               ← 上衣 x1
    41315    1               ← 下装 x1
    41624    1               ← 腰带 x1
    42092    1               ← 鞋子 x1
    2660673   6              ← 称号 x6
[/package data]
```

**示例**：`XBOX套 [鬼剑士]` — 打开直接获得整套7件装扮

---

### 3.5 `[usable cera package]` — 可使用礼包

**机制**：与 `[cera package]` 类似，获得全部物品，但通常有职业限制和技能选择功能。

```
[stackable type]
    `[usable cera package]`    0

[package data]
    601550043    1             ← 上衣 x1
    601560041    1             ← 下装 x1
    601570038    1             ← 头发 x1
    601520037    1             ← 帽子 x1
    601500042    1             ← 鞋子 x1
    601510042    1             ← 脸部 x1
    601530034    1             ← 胸部 x1
    601540042    1             ← 腰带 x1
[/package data]

[suitable job]
    `[swordman]`              ← 限定鬼剑士使用
[/suitable job]
```

**示例**：`庆典礼服时装宝箱[男鬼剑士]` — 各职业独立的完整8件套时装

**与 `[cera package]` 的区别**：
- `[cera package]`：直接打开，无限制
- `[usable cera package]`：有职业限制，可能涉及技能选择等附加逻辑

---

### 3.6 `[random upgradable legacy]` — 可升级随机宝箱

**机制**：最复杂的类型，使用 `[RANDOMBOX]` 结构，支持多轮随机、消耗材料、升级判定。

```
[stackable type]
    `[random upgradable legacy]`    1

[RANDOMBOX]
    [int data]                     ← 第1轮奖励池
        42    1        0           ← type=42, weight=1, count=0
        42    30000    1           ← type=42, weight=30000, count=1
        42    20000    2           ← type=42, weight=20000, count=2
        42    5000     3          ← type=42, weight=5000, count=3
        3     400      1
        24     25600    1
        ...
    [/int data]
    [int data]                     ← 第2轮奖励池
        2682154    5        0
        2682154    17000    3
        ...
        10004522    1000    1      ← 末尾flag=1表示该轮的升级物品
    [/int data]
    [int data]                     ← 第3轮奖励池
        43    1        0
        ...
    [/int data]
    [sealing removal item]         ← 开启消耗的物品
        2    10004520    3    10004517    1
    [/sealing removal item]
    [not enough item msg]          ← 材料不足提示
        `材料不足`
    [/not enough item msg]
[/RANDOMBOX]
```

**`[int data]` 格式**：`type/itemCode  probability  count  flag`
- `flag=0`：普通奖励
- `flag=1`：升级物品（抽中后进入下一轮）

**示例**：`马戏团解密礼盒` — 需要3个马戏团神秘钥匙开启，多轮随机+升级机制

---

## 四、六种类型对比总结

| 类型 | 类型标识 | 选择方式 | 获得数量 | 典型用途 |
|------|---------|---------|---------|---------|
| 多轮随机 | `[booster]` | 自动随机 | 每轮1个，共N轮 | 等级达成奖励、综合奖励盒 |
| 单轮随机 | `[booster random]` | 自动随机 | 1个 | 武器盒、称号盒、宠物蛋 |
| 自选礼盒 | `[booster selection]` | 玩家选择 | 1个 | 自选光环/宠物/武器/首饰 |
| 直开礼包 | `[cera package]` | 无 | 全部 | 完整套装礼包 |
| 可用礼包 | `[usable cera package]` | 无(可能有技能选) | 全部 | 职业限定时装套 |
| 可升级随机 | `[random upgradable legacy]` | 自动随机+升级 | 多轮 | 复杂活动宝箱 |

---

## 五、概率系统详解

### 5.1 权重机制

所有随机类型使用**权重(weight)**而非百分比：

```
单个物品概率 = 物品权重 / 该块所有物品权重之和
```

### 5.2 典型权重分布

**均匀分布**（来历不明的武器盒子）：
```
15个物品，每个权重5000 → 每个物品概率 ≈ 6.67%
```

**阶梯分布**（兔年幸运称号礼盒）：
```
普通称号: 权重4700 (各47%)   ← 容易获得
中级称号: 权重250  (各2.5%)  ← 较难获得
高级称号: 权重50   (各0.5%)  ← 极难获得
```

**极端分布**（解放之钥袋 - 点券数量）：
```
2个:  权重886295 (88.6%)     ← 最常见
5个:  权重7977   (0.8%)
100个: 权重9     (0.0009%)   ← 极稀有
```

---

## 六、完整 .stk 文件结构速查

```
#PVF_File

# ==================== 基础信息 ====================
[name]              `道具名称`
[name2]             `English Name`
[explain]           `功能描述`
[flavor text]       `背景故事`

# ==================== 属性 ====================
[grade]             1
[rarity]            2                          # 2=稀有, 3=史诗
[weight]            1
[price]             0
[stack limit]       1000
[cool time]         1000                       # 冷却时间(ms)
[minimum level]     1

# ==================== 职业与绑定 ====================
[usable job]        `[all]`                    # 可使用职业
[/usable job]
[attach type]       `[trade]`                  # 绑定: [trade]/[account]
[suitable job]      `[swordman]`               # 推荐职业(可选)
[/suitable job]

# ==================== 图标 ====================
[icon]              `Item/stackable/xxx.img`    13
[icon mark]         `Item/IconMark_cn.img`      75
[field image]       `Item/FieldImage.img`       30

# ==================== 礼盒核心 ====================
[stackable type]    `[礼盒类型]`    0           # ★ 关键：决定礼盒类型

# === [booster] / [booster random] 的奖励数据 ===
[booster info]
    [etc | equipment | cera]
        N                                       # 本块抽取数量
        itemCode    weight    count             # 物品列表
        ...
    [/etc | equipment | cera]
    ...                                         # [booster]可有多个块
[/booster info]

# === [booster selection] 的选择数据 ===
[booster category num]
    tabCount    itemCountPerTab
[booster selection num]
    totalSelectionsAllowed
[booster select category]
    tabIndex    subIndex
    [avatar | equipment | etc]
        itemCode    count    [param1]    [param2]
        ...
    [/avatar | equipment | etc]
[/booster select category]
...                                             # 多个标签页
[booster category name]
    `提示文字`
    `标签页1名称`
    `标签页2名称`
    ...
[/booster category name]

# === [cera package] / [usable cera package] 的物品列表 ===
[package data]
    itemCode    count
    ...
[/package data]

# === [random upgradable legacy] 的复杂随机 ===
[RANDOMBOX]
    [int data]
        type    weight    count    flag
        ...
    [/int data]
    ...
    [sealing removal item]
        ...    itemCode    ...    itemCode
    [/sealing removal item]
    [not enough item msg]
        `材料不足提示`
    [/not enough item msg]
[/RANDOMBOX]

# ==================== 附加设置 ====================
[need material]             itemCode    count   # 开启所需材料
[npc gift disallowance]     1                    # 禁止NPC赠送
[expiration date]           1374721200           # 过期时间戳
[random option unseal]      1                    # 装备随机封印属性
[move wav]                  `BONE_TOUCH`         # 拾取音效
```

---

## 七、新增礼盒实操指南

### 7.1 创建步骤总览

```
1. 确定礼盒类型 → 选择 [stackable type]
2. 创建 .stk 文件 → 填写基础属性
3. 注册到 stackable/stackable.lst → 分配ItemCode
4. 准备奖励物品 → 确认所有奖励物品的ItemCode存在
5. 设置图标资源 → 准备icon图并注册到.img文件
6. 测试验证 → 导入PVF后游戏内测试
```

### 7.2 创建一个自选礼盒（最常用）

**需求**：创建一个"新春武器自选礼盒"，玩家可以从鬼剑士/格斗家/神枪手三个职业中各选一把武器。

```ini
#PVF_File

[name]
    `新春武器自选礼盒`

[explain]
    `开启后，可以从鬼剑士、格斗家、神枪手的专属武器中选择获得1把。`

[grade]
    1

[rarity]
    3

[icon mark]
    `Item/IconMark_cn.img`    75

[usable job]
    `[all]`
[/usable job]

[attach type]
    `[account]`

[minimum level]
    1

[icon]
    `Item/stackable/cash_cn_2023.img`    13

[stackable type]
    `[booster selection]`    0

[booster category num]
    1    3

[booster selection num]
    1

# --- 鬼剑士武器页 ---
[booster select category]
    0    0
    [equipment]
        601000000    1
        601010000    1
        601020000    1
    [/equipment]
[/booster select category]

# --- 格斗家武器页 ---
[booster select category]
    1    0
    [equipment]
        602000000    1
        602010000    1
    [/equipment]
[/booster select category]

# --- 神枪手武器页 ---
[booster select category]
    2    0
    [equipment]
        604000000    1
        604010000    1
    [/equipment]
[/booster select category]

[booster category name]
    `请选择职业`
    `鬼剑士`
    `格斗家`
    `神枪手`
[/booster category name]

[move wav]
    `BONE_TOUCH`

[npc gift disallowance]
    1
```

### 7.3 创建一个随机礼盒

**需求**：创建一个"神秘宝箱"，有50%概率获得普通材料，30%概率获得稀有材料，20%概率获得称号。

```ini
#PVF_File

[name]
    `神秘宝箱`

[explain]
    `开启后，可以随机获得一种道具。`

[grade]
    1

[rarity]
    2

[icon mark]
    `item/iconmark.img`    64

[weight]
    1

[usable job]
    `[all]`
[/usable job]

[attach type]
    `[trade]`

[minimum level]
    1

[stack limit]
    100

[cool time]
    1000

[icon]
    `Item/stackable/consumption.img`    281

[stackable type]
    `[booster random]`    0

[booster info]
    [etc]
        1
        3035    3000    1          # 普通材料A, 权重3000, 概率30%
        3036    2000    1          # 普通材料B, 权重2000, 概率20%
        3033    2000    1          # 普通材料C, 权重2000, 概率20%
        690000363    2000    1     # 稀有材料, 权重2000, 概率20%
        2681090    1000    1       # 称号, 权重1000, 概率10%
    [/etc]
[/booster info]

[move wav]
    `BONE_TOUCH`

[npc gift disallowance]
    1
```

### 7.4 创建一个多轮奖励礼盒

**需求**：创建一个"冒险家成长礼盒"，开启后固定获得经验药水x3 + 随机获得一件装备 + 随机获得一个称号。

```ini
#PVF_File

[name]
    `冒险家成长礼盒`

[explain]
    `开启后，获得经验药水x3，并随机获得一件装备和一个称号。`

[grade]
    1

[rarity]
    3

[icon mark]
    `Item/IconMark_cn.img`    75

[usable job]
    `[all]`
[/usable job]

[attach type]
    `[account]`

[minimum level]
    1

[icon]
    `Item/stackable/cash_cn_2023.img`    14

[stackable type]
    `[booster]`    0

[cool time]
    1000

[booster info]
    [etc]                            # 第1轮: 固定经验药水
        1
        3330    10000    3           # 权重10000(100%), 数量3
    [/etc]
    [etc]                            # 第2轮: 随机装备
        1
        2242102    5000    1         # 装备A, 33.3%
        2270102    5000    1         # 装备B, 33.3%
        2271102    5000    1         # 装备C, 33.3%
    [/etc]
    [etc]                            # 第3轮: 随机称号
        1
        2681090    7000    1         # 普通称号, 70%
        2681094    3000    1         # 稀有称号, 30%
    [/etc]
[/booster info]

[move wav]
    `BONE_TOUCH`

[npc gift disallowance]
    1
```

### 7.5 创建一个完整套装礼包

**需求**：创建一个"限定时装礼包"，打开直接获得8件时装。

```ini
#PVF_File

[name]
    `限定时装礼包`

[explain]
    `开启后，直接获得一整套限定时装（8件）。`

[grade]
    1

[rarity]
    3

[attach type]
    `[account]`

[icon mark]
    `Item/IconMark_cn.img`    75

[usable job]
    `[all]`
[/usable job]

[minimum level]
    1

[icon]
    `Item/stackable/cash_cn_2023.img`    13

[stackable type]
    `[cera package]`    0

[suitable job]
    `[swordman]`
[/suitable job]

[package data]
    601550043    1              # 上衣
    601560041    1              # 下装
    601570038    1              # 头发
    601520037    1              # 帽子
    601500042    1              # 鞋子
    601510042    1              # 脸部
    601530034    1              # 胸部
    601540042    1              # 腰带
[/package data]

[move wav]
    `BONE_TOUCH`

[npc gift disallowance]
    1
```

---

## 八、注意事项与常见问题

### 8.1 ItemCode 分配规则

- ItemCode 必须在对应的 LST 文件中已注册
- 不同类型的物品代码范围不同（装备、消耗品、称号等各有区间）
- 自定义物品需要确保 ItemCode 不与现有物品冲突

### 8.2 图标资源

- `[icon]` 字段引用 `.img` 精灵图 + 索引号
- 图标精灵图通常位于 `Item/stackable/` 目录下
- 自定义图标需要修改对应的 `.img` 文件

### 8.3 `[attach type]` 绑定类型选择

| 类型 | 说明 |
|------|------|
| `[trade]` | 可自由交易 |
| `[account]` | 帐号绑定，可通过帐号金库转移 |

### 8.4 booster info 中的物品类型标签

| 标签 | 说明 | 条目格式 |
|------|------|---------|
| `[etc]` | 消耗品/材料/任务物品 | `itemCode weight count` |
| `[equipment]` | 装备/称号 | `itemCode weight count` |
| `[cera]` | 点券/货币 | `type weight count` |
| `[avatar]` | 装扮/光环（仅选择类） | `itemCode count param1 param2` |

### 8.5 `[booster category num]` 参数解读

```
[booster category num]
    A    B
```

- **A**：玩家总共可选择的物品数量（通常为1）
- **B**：标签页的数量（每个标签页对应一个 `[booster select category]`）

特殊情况：
- `0` 表示不使用标签页分类，所有物品在同一列表中
- `1 1` 表示1个标签页，选1件物品

---

## 九、附录：实际礼盒文件索引

### 按类型分类的代表文件

| 类型 | 文件路径 | 名称 |
|------|---------|------|
| booster | `stackable/twdf/lv70/chn_box_sm.stk` | LV.70达成礼盒 |
| booster | `stackable/twdf/event/130711_event/ontime/1_ontimebox.stk` | OnTime礼物袋 |
| booster | `stackable/twdf/event/130711_event/creator_event/creator_eqipbox_10lv.stk` | 10等级装备箱 |
| booster (cera) | `stackable/twdf/event/130711_event/ontime/2_keybox.stk` | 解放之钥袋 |
| booster random | `stackable/twdf/mohedaoju/wuqihezi.stk` | 武器盒子 |
| booster random | `stackable/twdf/mohedaoju/shoushihezi.stk` | 首饰盒子 |
| booster random | `stackable/twdf/tianjia/2011_title_box1.stk` | 称号礼盒 |
| booster random | `stackable/twdf/tianjia/2011_creature_box1.stk` | 宠物礼盒 |
| booster selection | `stackable/cash/2023newyear/chn_490808008.stk` | 光环自选礼盒 |
| booster selection | `stackable/cash/2023newyear/chn_490808010.stk` | 宠物自选礼盒 |
| booster selection | `stackable/twdf/event/130328_growthweapon/130328_growthweaponbox.stk` | 成长武器箱 |
| booster selection | `stackable/twdf/event/130711_event/ontime/4_accbox.stk` | 饰品宝箱 |
| cera package | `stackable/cash/package_xboxavatar_sm.stk` | XBOX套礼包 |
| usable cera package | `stackable/twdf/event/130711_event/festival_event/2_dressbox2_slayer.stk` | 时装宝箱 |
| random upgradable | `stackable/event/chn_event_20150115_circus_dungeon/renewal/chn_circusbox_10004517.stk` | 马戏团礼盒 |

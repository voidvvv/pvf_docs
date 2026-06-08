# ATSwordman 角色渲染系统分析

## 一、整体架构概览

角色渲染由三个核心层组成：

```
.chr (角色定义) --> .ani (动画帧) --> .img/.spr (精灵图片)
                -->
     .equ (装备定义) --> .lay (层级脚本) --> .ani --> .img/.spr
```

### 关键文件路径

| 文件类型 | 路径 |
|---------|------|
| 角色定义 | `character/swordman/atswordman.chr` |
| 角色列表 | `character/character.lst`（ItemCode=10） |
| 动画文件 | `character/swordman/atanimation/*.ani` |
| 层级脚本 | `equipment/character/atswordman.lay` |
| 身体图片 | `Character/swordman/ATEquipment/Avatar/skin/sg_body%04d.img` |
| 装备目录 | `equipment/character/swordman/at_avatar/{skin,hair,coat,pants,...}/` |

### 普通Swordman vs ATSwordman 的区别

| 项目 | Swordman | ATSwordman |
|------|----------|------------|
| 身体图片路径 | `Equipment/Avatar/skin/sm_body%04d.img` | `ATEquipment/Avatar/skin/sg_body%04d.img` |
| 动画目录 | `Animation/` | `ATAnimation/` |
| 层级脚本 | `swordman.lay` | `atswordman.lay` |
| 职业标识 | `[swordman]` | `[creator mage]` |

---

## 二、动画整体渲染逻辑

### 2.1 角色定义文件 (.chr)

`atswordman.chr` 定义了角色的全部基础配置：

```
[job]
    `[creator mage]`          ← 职业标识

[growtype name]
    `鬼剑士`                   ← 转职名称列表
    `驭剑士`
    `流浪武士`
    `契魔者`
    `暗殿骑士`
    `鬼剑士`

[body image path]
    `Character/swordman/ATEquipment/Avatar/skin/sg_body%04d.img`  ← 身体图片模板

[waiting motion]
    `ATAnimation/Stay.ani`    ← 待机动画

[attack motion]
    `ATAnimation/Attack1.ani` ← 攻击动画（多段）
    `ATAnimation/Attack2.ani`
    `ATAnimation/Attack3.ani`
    `ATAnimation/Attack4.ani`

[etc motion]
    ...技能动画列表...
```

**chr文件还包含**：基础属性值（HP/MP/攻击力等）、各转职成长属性、觉醒名称、技能列表、武器音效、武器命中信息、武器技能系数等。

### 2.2 动画文件 (.ani) 结构

以 `ATAnimation/Stay.ani` 为例：

```
[LOOP]
    1                         <-- 是否循环（1=循环）

[FRAME MAX]
    4                         <-- 总帧数

[FRAME000]
    [IMAGE]
        `character/swordman/atequipment/avatar/skin/sg_body%04d.img`
        9                     <-- 帧索引号（%04d被替换为0009）
    [IMAGE POS]
        -248  -379            <-- 图片绘制偏移坐标(x, y)
    [DELAY]
        800                   <-- 帧延迟（毫秒）
    [DAMAGE BOX]
        -29  -5  -6  50  10  112  <-- 受击判定框参数

[FRAME001]
    [IMAGE]
        `character/swordman/atequipment/avatar/skin/sg_body%04d.img`
        10                    <-- 第二帧使用索引10 → sg_body0010.img
    [IMAGE POS]
        -248  -379
    [DELAY]
        200
    ...
```

以 `ATAnimation/Attack1.ani` 为例（攻击动画更复杂）：

```
[FRAME MAX]
    6

[FRAME000]
    [IMAGE]
        `Character/swordman/ATEquipment/Avatar/skin/sg_body%04d.img`
        48
    [IMAGE POS]
        -248  -379
    [DELAY]
        80
    [DAMAGE BOX]              <-- 受击判定框（多个）
        -28  -5  30  40  10  40
        ...
    [ATTACK BOX]              <-- 攻击判定框（从FRAME001开始出现）
        -56  -15  39  57  30  39
        ...
```

### 2.3 .ani 文件中的关键元素

| 元素 | 说明 |
|------|------|
| `[LOOP]` | 是否循环播放（1=循环，用于待机/移动等持续性动画） |
| `[FRAME MAX]` | 总帧数 |
| `[IMAGE]` | 图片路径模板 + 帧索引号（占位符 `%04d` 被索引号替换） |
| `[IMAGE POS]` | 图片绘制偏移坐标（x, y），用于将精灵图定位到角色中心 |
| `[DELAY]` | 帧延迟时间（毫秒），控制播放速度 |
| `[DAMAGE BOX]` | 受击判定框，定义角色被攻击时的碰撞区域 |
| `[ATTACK BOX]` | 攻击判定框，定义角色攻击时的有效范围（仅攻击动画有） |

### 2.4 动画播放流程

```
1. 游戏引擎根据角色当前状态选择动画
   (待机 → Stay.ani, 攻击 → Attack1.ani, 移动 → Move.ani, ...)

2. 读取动画文件的帧数据

3. 逐帧渲染:
   a. 将 [IMAGE] 中的 %04d 替换为帧索引号
      → sg_body%04d.img + 索引9 = sg_body0009.img
   b. 在 [IMAGE POS] 指定的偏移位置绘制该帧精灵图
   c. 等待 [DELAY] 毫秒
   d. 如果 [LOOP]=1，循环回第一帧；否则播放完毕

4. 同步渲染装备图层（见下节）
```

---

## 三、身体部位（装备）渲染逻辑

### 3.1 装备槽位系统

ATSwordman的装备目录位于 `equipment/character/swordman/at_avatar/`，按部位分为以下槽位：

| 目录 | 装备类型标识 | 说明 |
|------|-------------|------|
| `skin/` | `[skin avatar]` | 皮肤（全身替换型） |
| `hair/` | `[hair avatar]` | 头发 |
| `hat/` | `[hat avatar]` | 帽子 |
| `face/` | `[face avatar]` | 脸部 |
| `breast/` | `[breast avatar]` | 胸部装饰 |
| `coat/` | `[coat avatar]` | 上衣 |
| `pants/` | `[pants avatar]` | 下装 |
| `waist/` | `[waist avatar]` | 腰带 |
| `shoes/` | `[shoes avatar]` | 鞋子 |

### 3.2 装备文件 (.equ) 结构

以默认上衣 `equipment/character/swordman/at_avatar/coat/acoat_default.equ` 为例：

```
[name]
    `默认上衣`

[usable job]
    `[creator mage]`          <-- 限定职业
[/usable job]

[equipment type]
    `[coat avatar]`    0      <-- 装备槽位类型

[animation job]
    `[creator mage]`          <-- 动画职业标识

[variation]
    0    0                    <-- 变体ID（0=默认）

[layer variation]
    2300                      <-- 层级优先级（数值越大越后渲染=显示在越上层）
    `coat_c`                  <-- 部件文件夹名
[equipment ani script]
    `equipment/character/atswordman.lay`  <-- 该图层的动画映射脚本

[layer variation]
    1800
    `coat_a`
[equipment ani script]
    `equipment/character/atswordman.lay`

[layer variation]
    900
    `coat_b`
[equipment ani script]
    `equipment/character/atswordman.lay`

[layer variation]
    500
    `coat_d`
[equipment ani script]
    `equipment/character/atswordman.lay`
```

**关键发现：每件装备可以有多个图层，分布在不同渲染深度。**

### 3.3 默认装备的图层分析

#### 默认头发 (`ahair_dafault.equ`)

```
[layer variation]  2000   `hair_a`      <-- 头发主体（最外层）
[layer variation]   800   `hair_b`      <-- 头发背面
[layer variation]   400   `hair_d`      <-- 头发细节（最内层）
```

#### 默认上衣 (`acoat_default.equ`)

```
[layer variation]  2300   `coat_c`      <-- 上衣背面/披风（最外层）
[layer variation]  1800   `coat_a`      <-- 上衣主体
[layer variation]   900   `coat_b`      <-- 上衣次要层
[layer variation]   500   `coat_d`      <-- 上衣细节（最内层）
```

#### 默认下装 (`apants_default.equ`)

```
[layer variation]  1500   `pants_a`     <-- 下装主体
[layer variation]  1300   `pants_b`     <-- 下装次要层
```

### 3.4 层级动画脚本 (.lay)

`equipment/character/atswordman.lay` 是装备图层的核心动画映射文件：

```
[LAYER]
    0                          <-- 层级ID

[waiting motion]
    `%s/Stay.ani`             <-- %s 在运行时被替换为部件文件夹路径

[move motion]
    `%s/Move.ani`

[attack motion]
    `%s/Attack1.ani`
    `%s/Attack2.ani`
    `%s/Attack3.ani`
    `%s/Attack4.ani`

[etc motion]
    `%s/LiftSlash.ani`
    `%s/DarkSlash1.ani`
    ...数百个技能动画映射...
```

**`%s` 占位符替换机制**：

当渲染装备的 `coat_a` 图层时：
1. 读取 `.lay` 中的动画路径 `%s/Stay.ani`
2. 将 `%s` 替换为 `coat_a` → 得到 `coat_a/Stay.ani`
3. 加载该动画文件，其中引用 `coat_a` 文件夹下的精灵图

同理，`hair_a` 图层 → `hair_a/Stay.ani` → `hair_a` 文件夹下的精灵图。

**这意味着每个装备部件文件夹下都有完整的一套动画文件**，确保每个身体部位在所有动作中都能正确显示。

### 3.5 渲染排序（从底层到顶层）

根据默认装备的层级优先级，完整的渲染顺序为：

```
优先级    部件           说明
──────────────────────────────────────────────
  400    hair_d        头发细节（最底层，最先渲染）
  500    coat_d        上衣细节
  800    hair_b        头发背面
  900    coat_b        上衣次要层
 1300    pants_b       下装次要层
 1500    pants_a       下装主体
 1800    coat_a        上衣主体
 2000    hair_a        头发主体
 2300    coat_c        上衣背面/披风（最顶层，最后渲染）
──────────────────────────────────────────────
↑ 先渲染（被遮挡）              后渲染（覆盖在上层） ↑
```

**排序规则**：
- 所有已装备物品的 `[layer variation]` 优先级值收集到一起
- 按数值从小到大排序
- 依次渲染：数值小的先画（被后面的图层覆盖），数值大的后画（显示在前面）

**设计精妙之处**：同一件装备的多个图层分布在不同深度，与其他部位的图层交错排列。例如头发有3层分别位于400/800/2000，上衣有4层分别位于500/900/1800/2300，这样头发的一部分会在衣服前面（2000>1800），另一部分在衣服后面（800<900），实现了自然的遮挡效果。

---

## 四、皮肤Avatar（全身替换机制）

### 4.1 皮肤装备结构

以 `equipment/character/swordman/at_avatar/skin/201807.equ`（兔女郎COS皮肤）为例：

```
[equipment type]
    `[skin avatar]`    0      <-- 皮肤类型（非部件类型）

[variation]
    0    2022                  <-- 变体ID=2022，决定使用哪套精灵图

[hide equipment]               <-- 隐藏其他所有部位装备
    `[hat avatar]`
    `[hair avatar]`
    `[face avatar]`
    `[breast avatar]`
    `[coat avatar]`
    `[pants avatar]`
    `[waist avatar]`
    `[shoes avatar]`

[hide layer]                   <-- 隐藏特定层级ID的所有图层
    2310  1961  515  1840  ...（大量层级ID）

[hide grow avatar]             <-- 隐藏特定转职特效装备
    `Soul Bringer Aura`
    `Weapon Master Gloves`
    ...
```

### 4.2 皮肤Avatar的工作方式

1. `[variation] 0 2022` → 变体ID改变基础身体图片的精灵图选择
2. `[hide equipment]` → 隐藏帽子、头发、脸、衣服、裤子、腰带、鞋子等所有部位
3. `[hide layer]` → 通过层级ID精确隐藏额外的装饰图层
4. `[hide grow avatar]` → 隐藏各转职的专属外观效果

**结果**：皮肤Avatar完全替换角色外观，不叠加任何其他装备。

---

## 五、完整渲染流程总结

```
┌─────────────────────────────────────────────────────────┐
│                    角色渲染完整流程                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 加载角色定义                                         │
│     atswordman.chr → 读取基础属性、动画映射、图片模板       │
│                           ↓                             │
│  2. 确定当前动作状态                                      │
│     待机/移动/攻击/受击/技能/跳跃...                        │
│                           ↓                             │
│  3. 加载对应动画文件                                      │
│     ATAnimation/Stay.ani → 读取帧数据                     │
│     （帧数、图片索引、偏移、延迟、碰撞框）                    │
│                           ↓                             │
│  4. 渲染基础身体层                                        │
│     sg_body%04d.img + 帧索引 → sg_body0009.img           │
│     使用 IMAGE POS 定位绘制                               │
│                           ↓                             │
│  5. 遍历所有已装备的 .equ 文件                             │
│     读取每个装备的 [layer variation] 列表                   │
│     每个图层: 优先级数值 + 部件文件夹名                     │
│                           ↓                             │
│  6. 收集所有图层并按优先级排序                              │
│     从小到大排列                                          │
│                           ↓                             │
│  7. 按排序逐层渲染                                        │
│     对每个图层:                                           │
│       部件文件夹名 + .lay动画路径 → 具体.ani文件            │
│       .ani引用该部件的.img精灵图                           │
│       叠加绘制到画面上                                     │
│                           ↓                             │
│  8. 合成最终角色画面                                      │
│     基础身体 + 所有装备图层 = 完整角色外观                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 六、附录：数据结构速查

### .chr 关键字段

| 字段 | 用途 |
|------|------|
| `[job]` | 职业标识 |
| `[growtype name]` | 各转职名称 |
| `[body image path]` | 基础身体图片路径模板 |
| `[waiting/move/attack/... motion]` | 基础动画文件路径 |
| `[etc motion]` | 扩展技能动画列表 |
| `[attack info]` / `[etc attack info]` | 攻击判定数据文件路径 |
| `[weapon wav]` | 武器音效 |
| `[weapon hit info]` | 武器命中效果 |
| `[weapon skill info]` | 武器技能系数 |

### .equ 关键字段

| 字段 | 用途 |
|------|------|
| `[equipment type]` | 装备槽位类型 |
| `[usable job]` | 可使用职业限制 |
| `[animation job]` | 动画职业标识 |
| `[variation]` | 变体ID |
| `[layer variation]` | 图层优先级 + 部件文件夹名（可多个） |
| `[equipment ani script]` | 图层动画映射脚本(.lay文件) |
| `[hide equipment]` | 装备时隐藏的其他槽位 |
| `[hide layer]` | 装备时隐藏的特定层级ID |
| `[hide grow avatar]` | 装备时隐藏的转职外观 |
| `[icon]` | 物品图标 |
| `[avatar select ability]` | 可选择的附魔属性 |

### .lay 关键字段

| 字段 | 用途 |
|------|------|
| `[LAYER]` | 层级ID |
| `[waiting/move/attack/... motion]` | 各动作的动画路径模板（`%s` 占位符） |

### .ani 关键字段

| 字段 | 用途 |
|------|------|
| `[LOOP]` | 是否循环 |
| `[FRAME MAX]` | 总帧数 |
| `[IMAGE]` | 图片路径模板 + 帧索引号 |
| `[IMAGE POS]` | 绘制偏移(x, y) |
| `[DELAY]` | 帧延迟(ms) |
| `[DAMAGE BOX]` | 受击判定框 |
| `[ATTACK BOX]` | 攻击判定框 |

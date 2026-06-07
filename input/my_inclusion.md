# dnf中的nut

## 是什么
nut是dnf pvf封包中的一种脚本，其用Squirrel脚本编写，文件后缀为.nut，是dnf客户端与服务端共同解析使用的，使用各种钩子函数对游戏进行扩展的脚本。
所有nut脚本均存放于pvf中的sqr目录下。需要使用pvfutility mcp读取pvf解包数据后，再去sqr目录下获取nut脚本

## 能做什么
nut能做的非常多，比较常见的就是给角色新增常驻状态，更新技能形态，甚至新增技能！

---

## PVF目录结构概览

PVF封包的根目录包含以下关键目录：

| 目录 | 说明 |
|------|------|
| `sqr/` | 所有NUT脚本的根目录 |
| `appendage/` | .apd格式的appendage定义文件 |
| `passiveobject/` | 被动对象的.obj定义文件 |
| `character/` | 角色相关资源（动画、技能等） |
| `equipment/` | 装备定义文件 |
| `skill/` | 技能定义文件 |
| `monster/` | 怪物相关资源 |
| `map/` | 地图相关资源 |
| `stackable/` | 消耗品等可堆叠物品 |
| `npc/` | NPC相关资源 |

---

## SQR目录结构与NUT脚本分类

`sqr/` 目录下按功能和角色组织NUT脚本，主要结构如下：

```
sqr/
├── character/                    # 角色相关脚本（按职业分类）
│   ├── common/                   # 公共脚本（如burster、jump等通用状态）
│   ├── swordman/                 # 鬼剑士
│   ├── fighter/                  # 格斗家
│   ├── gunner/                   # 神枪手
│   ├── mage/                     # 魔法师
│   ├── priest/                   | 圣职者
│   ├── thief/                    # 暗夜使者
│   ├── atmage/                   # 魔法师（转职后）
│   ├── atswordman/               # 鬼剑士（转职后）
│   ├── atfighter/                # 格斗家（转职后）
│   ├── atgunner/                 # 神枪手（转职后）
│   ├── atpriest/                 # 圣职者（转职后）
│   ├── demonicswordman/          # 鬼剑士（三次觉醒？未确认）
│   └── {character}_load_state.nut # 各角色的状态注册文件
├── appendage/character/          # 共享appendage脚本
├── monster/                      # 怪物相关脚本
├── map/                          # 地图相关脚本
├── equipment/                    # 装备相关脚本
└── custom/                       # 自定义内容
```

每个角色目录下的子目录结构通常是：

```
sqr/character/{职业}/
├── {职业}_header.nut              # 常量定义（STATE_*, SKILL_*, CUSTOM_ANI_*等）
├── {职业}_common.nut              # 公共函数
├── {技能名}/                      # 每个技能一个目录
│   ├── {技能名}.nut               # 技能状态脚本
│   ├── po_{被动对象名}.nut         # 被动对象脚本（如有）
│   └── ap_{appendage名}.nut       # appendage脚本（如有）
├── appendage/                     # 该角色的appendage脚本
│   └── ap_{appendage名}.nut
├── {职业}_load_state.nut          # 状态注册入口（在sqr/character/下）
```

### NUT脚本文件类型

| 类型 | 命名规则 | 说明 |
|------|---------|------|
| Header文件 | `{职业}_header.nut` | 定义STATE_*, SKILL_*, CUSTOM_ANI_*等枚举常量 |
| Load State文件 | `{职业}_load_state.nut` | 注册脚本文件、技能状态和被动对象 |
| 技能状态文件 | `{技能名}.nut` | 定义技能的行为逻辑（状态切换、动画、创建被动对象等） |
| 被动对象文件 | `po_{名称}.nut` | 定义被动对象的行为（弹道、特效区域等） |
| Appendage文件 | `ap_{名称}.nut` | 定义appendage（状态效果）的行为 |
| 被动技能文件 | `passive_skill_{职业}.nut` | 定义被动技能如何创建和附加appendage |
| 公共函数文件 | `{职业}_common.nut` | 定义该职业共享的工具函数 |

---

## Load State文件（状态注册入口）

每个角色有一个 `{职业}_load_state.nut` 文件，是NUT脚本的注册入口。此文件负责：

### 1. 加载脚本文件
```squirrel
// 加载header（常量定义）
IRDSQRCharacter.pushScriptFiles("Character/ATMage/atmage_header.nut");
// 加载公共函数
IRDSQRCharacter.pushScriptFiles("Character/ATMage/atmage_common.nut");
// 加载被动技能处理
IRDSQRCharacter.pushScriptFiles("Character/ATMage/passive_skill_ATMage.nut");
```

### 2. 注册技能状态
```squirrel
// 参数：职业枚举ID, 脚本路径, 状态名称字符串, STATE常量, SKILL常量
// STATE常量和SKILL常量在header文件中定义
// 如果SKILL参数为-1，表示该状态不绑定特定技能（如普通攻击、站立等基础状态）
IRDSQRCharacter.pushState(ENUM_CHARACTERJOB_AT_MAGE, "Character/ATMage/FireRoad/fire_road.nut", "FireRoad", STATE_FIRE_ROAD, SKILL_FIRE_ROAD);
```

### 3. 注册被动对象
```squirrel
// 参数：被动对象脚本路径, 被动对象唯一ID
// 这个ID必须与passiveobject/passiveobject.lst中注册的.obj文件ID对应
IRDSQRCharacter.pushPassiveObj("Character/ATMage/FireRoad/po_fire_road.nut", 24212);
// 同一个脚本可以注册多个ID（用于同一技能创建多个同类被动对象）
IRDSQRCharacter.pushPassiveObj("Character/ATMage/FireRoad/po_fire_road.nut", 24213);
```

### 4. 运行独立脚本
```squirrel
// 运行不绑定到特定角色的脚本
sq_RunScript("js60_qq506807329/js60_506807329_common.nut");
```

---

## 技能状态脚本（Skill State Script）

技能状态脚本定义了角色释放技能时的完整行为流程。每个技能通常对应一个.nut文件，以固定格式定义一系列回调函数：

### 函数列表

| 函数名格式 | 说明 |
|-----------|------|
| `checkExecutableSkill_{名称}(obj)` | 检查是否可以使用该技能（检测技能输入） |
| `checkCommandEnable_{名称}(obj)` | 检查当前状态下是否允许使用该技能指令 |
| `onSetState_{名称}(obj, state, datas, isResetTimer)` | 进入该状态时调用，设置子状态、播放动画 |
| `onEndCurrentAni_{名称}(obj)` | 当前动画播放完毕时调用，通常用于切换子状态或回到站立 |
| `onKeyFrameFlag_{名称}(obj, flagIndex)` | 动画关键帧触发时调用，常用于在特定帧创建被动对象 |
| `prepareDraw_{名称}(obj)` | 绘制前调用 |
| `onEndState_{名称}(obj, new_state)` | 离开该状态时调用，用于清理（如结束蓄力条） |

### 示例流程（以FireRoad为例）

```
1. 玩家按下技能键
   -> checkExecutableSkill_FireRoad() 检测到技能使用
   -> 切换到 STATE_FIRE_ROAD 状态，子状态0（施法阶段）

2. onSetState_FireRoad() 被调用
   -> 设置施法动画 CUSTOM_ANI_FIRE_ROAD_CAST1
   -> 开始蓄力条显示

3. onEndCurrentAni_FireRoad() 动画播放完
   -> 子状态0 -> 切换到子状态1（释放阶段）

4. onKeyFrameFlag_FireRoad() 关键帧触发
   -> 在关键帧处通过 obj.sq_SendCreatePassiveObjectPacket() 创建被动对象
   -> 每个被动对象由passiveobject.lst中的ID标识

5. 子状态1动画播放完 -> 回到站立状态 STATE_STAND
```

### 创建被动对象的关键API
```squirrel
// 写入自定义数据（发送给被动对象的setCustomData函数）
obj.sq_StartWrite();
obj.sq_WriteWord(pauseTime);       // 写入word(2字节)
obj.sq_WriteDword(damage1);        // 写入dword(4字节)
obj.sq_WriteByte(maxHit);          // 写入byte(1字节)

// 发送创建被动对象数据包
// 参数：被动对象ID(在passiveobject.lst中注册), 子状态, x偏移, y偏移, z偏移
obj.sq_SendCreatePassiveObjectPacket(passiveObjectIndex, 0, xPos, yPos, zPos);

// 或使用位置版本
sq_SendCreatePassiveObjectPacketPos(mage, passiveObjectIndex, 0, x, y, z);
```

---

## 被动对象系统（Passive Object）

### 概念
被动对象（Passive Object）是游戏中独立于角色的对象，如弹道、特效区域、召唤物等。每个被动对象由两部分组成：
1. **.obj文件**（在`passiveobject/`目录下）- 定义视觉和碰撞属性
2. **.nut脚本**（在`sqr/`目录下）- 定义行为逻辑

### 注册流程
1. 在`passiveobject/passiveobject.lst`中为.obj文件分配唯一数字ID
2. 在load_state文件中通过`IRDSQRCharacter.pushPassiveObj()`注册.nut脚本与ID的对应关系
3. 在技能脚本中通过`obj.sq_SendCreatePassiveObjectPacket(ID, ...)`创建

### .obj文件格式
被动对象的.obj文件定义了其基础属性，存放于`passiveobject/`目录下，示例如下：

```
#PVF_File

[floating height]
	1                          // 浮空高度

[layer]
	`[bottom]`                 // 渲染层级

[pass type]
	`[pass all]`               // 穿透类型

[piercing power]
	1000                       // 穿透力

[basic motion]
	`Animation/ATFireRoad/ATFireRoad1.ani`  // 基础动画

[attack info]
	`AttackInfo/ATFireRoad1.atk`            // 攻击判定信息

[etc attack info]                         // 附加攻击判定（可多阶段攻击）
	`AttackInfo/ATFireRoad2.atk`
[/etc attack info]

[name]
	`火焰爆炸`                 // 对象名称
```

### passiveobject.lst格式
lst文件将数字ID映射到.obj文件路径：
```
ID -> passiveobject/character/mage/atfireroad1.obj  "火焰爆炸"
```
例如：ID 24212 对应 `passiveobject/character/mage/atfireroad1.obj`

### 被动对象NUT脚本回调函数

| 函数名格式 | 说明 |
|-----------|------|
| `setCustomData_po_{名称}(obj, receiveData)` | 创建时调用，从receiveData读取技能传来的自定义数据 |
| `procAppend_po_{名称}(obj)` | 每帧调用，处理持续逻辑 |
| `onAttack_po_{名称}(obj, damager, boundingBox, isStuck)` | 命中目标时调用 |
| `onEndCurrentAni_po_{名称}(obj)` | 动画播放完毕时调用，通常用于销毁对象 |
| `onKeyFrameFlag_po_{名称}(obj)` | 关键帧触发时调用 |

### setCustomData读取数据示例
技能脚本中写入的数据按顺序通过receiveData读取：
```squirrel
// 技能脚本中写入（创建时）：
obj.sq_StartWrite();
obj.sq_WriteWord(pauseTime);     // 第1个：word
obj.sq_WriteDword(damage1);      // 第2个：dword
obj.sq_WriteDword(damage2);      // 第3个：dword
obj.sq_WriteByte(maxHit);        // 第4个：byte

// 被动对象脚本中读取：
function setCustomData_po_ATFireRoad1(obj, receiveData)
{
    local pauseTime = receiveData.readWord();    // 读word
    local damage1 = receiveData.readDword();     // 读dword
    local damage2 = receiveData.readDword();     // 读dword
    local maxHit = receiveData.readByte();       // 读byte
    // ...
}
```

### 销毁被动对象
```squirrel
function onEndCurrentAni_po_ATFireRoad1(obj)
{
    if(!obj) return;
    if(obj.isMyControlObject())
    {
        sq_SendDestroyPacketPassiveObject(obj);  // 发送销毁数据包
    }
}
```

---

## 基础概念

### appendage
简称ap，是dnf pvf中的一个概念。可以理解为给游戏内角色（怪物或者玩家角色）上的一个状态。具体的状态可以编写各种各样的状态：比如霸体，提升属性等等，甚至可以ap本身无效果（dummy），由别的地方来读取ap进行各种判定！

ap定义的方式有两种：
1. 在目录`appendage/`下，.apd格式的文件。这种方式定义的文件需要在`appendage/appendage.lst`中为其定义一个唯一的id，这种方式定义的appendage可以在装备中以`[appendage]`标签通过id来进行引用，可以达成穿戴某个装备时，某个事件触发时进行appendage判定或者赋予
2. 在sqr中，使用.nut文件定义，在nut中，ap的实现类似于挂载了监听游戏事件的钩子函数，当赋予了ap后，特定事件触发后会触发特定的ap方法。

### appendage .apd文件格式

.apd文件是声明式的appendage定义，存放在`appendage/`目录下，格式如下：

```
#PVF_File

[type]
	`change status`            // appendage类型

[duration]
	180000                     // 持续时间（毫秒），0表示永久

[effect animation]
	``                         // 效果动画路径

[buff]
	1                          // 是否为增益（1=增益，0=减益）

[icon image]
	``	0                      // 图标路径	图标索引

[max overlap]
	1                          // 最大叠加层数

[string data]                 // 字符串参数（按类型不同而不同）
	`physical attack`
	`magical attack`
	`attack speed`
	`move speed`
	`cast speed`
[/string data]

[int data]                    // 整数参数
	0	0	0	0	0
[/int data]

[float data]                  // 浮点参数（与string data一一对应，为各属性的修改值）
	100.0	100.0	10.0	10.0	15.0
[/float data]
```

#### .apd文件类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `change status` | 修改角色属性值 | 物理攻击、魔法攻击、攻速等 |
| `change basic attack type` | 改变基础攻击类型 | 将普攻变为魔法攻击 |
| `attack type` | 改变攻击属性 | 暗属性攻击等 |
| `dummy` | 无实际效果，作为标记使用 | 用于判定是否拥有某状态 |

**`change status`类型**：string data指定要修改的属性名，float data指定修改值。例如：
- string: `physical attack` / float: `100.0` 表示增加100点物理攻击
- string: `attack speed` / float: `10.0` 表示增加10%攻击速度

**`dummy`类型**：不修改任何属性，仅作为标记存在，其他脚本可以检测角色是否拥有此appendage来进行逻辑判定。

### appendage在NUT中的创建（通过被动技能）

被动技能文件（如`passive_skill_atmage.nut`）定义了被动技能如何创建和附加appendage：

```squirrel
// 检测是否已存在该appendage
if (!CNSquirrelAppendage.sq_IsAppendAppendage(obj, "character/atmage/stickmaster/ap_stickmaster.nut"))
{
    // 创建appendage，参数：目标对象, 来源对象, 技能索引, 是否增益, 脚本路径, 是否启用
    local appendage = CNSquirrelAppendage.sq_AppendAppendage(obj, obj, skill_index, false,
        "character/atmage/stickmaster/ap_stickmaster.nut", true);

    // 添加属性修改（changeStatus）
    local change_appendage = appendage.sq_AddChangeStatus("唯一标识名", obj, obj, 0,
        CHANGE_STATUS_TYPE_PHYSICAL_ATTACK_BONUS, true, atkbonus);

    // 清空并重新设置参数
    change_appendage.clearParameter();
    change_appendage.addParameter(CHANGE_STATUS_TYPE_ATTACK_SPEED, true, spdbonus);
    change_appendage.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_CRITICAL_HIT_RATE, false, cthbonus);
}

// 移除appendage
CNSquirrelAppendage.sq_RemoveAppendage(obj, "character/atmage/stickmaster/ap_stickmaster.nut");
```

### appendage在sqr中的NUT脚本定义

在sqr中，ap的实现类似于挂载了监听游戏事件的钩子函数。NUT脚本定义的appendage必须按照指定格式：

```squirrel
// ===== 必须定义的第一个函数：注册钩子 =====
function sq_AddFunctionName(appendage)
{
    appendage.sq_AddFunctionName("proc", "proc_appendage_atmage_effect")
    appendage.sq_AddFunctionName("prepareDraw", "prepareDraw_appendage_atmage_effect")
    appendage.sq_AddFunctionName("onStart", "onStart_appendage_atmage_effect")
    appendage.sq_AddFunctionName("onEnd", "onEnd_appendage_atmage_effect")
    appendage.sq_AddFunctionName("drawAppend", "drawAppend_appendage_atmage_effect")
    appendage.sq_AddFunctionName("isEnd", "isEnd_appendage_atmage_effect")
}

// ===== 可选：添加前置效果 =====
function sq_AddEffect(appendage)
{
    //appendage.sq_AddEffectFront("Character/Priest/Effect/Animation/ScytheMastery/1_aura_normal.ani")
}

// ===== 各钩子回调函数实现 =====

// 每帧调用（核心逻辑通常在这里）
function proc_appendage_atmage_effect(appendage)
{
    if(!appendage) {
        return;
    }
}

// appendage被附加时调用
function onStart_appendage_atmage_effect(appendage)
{
    if(!appendage) {
        return;
    }
    local obj = appendage.getParent();
}

// 绘制前调用
function prepareDraw_appendage_atmage_effect(appendage)
{
    if(!appendage) {
        return;
    }
    local obj = appendage.getParent();
}

// appendage被移除时调用
function onEnd_appendage_atmage_effect(appendage)
{
    if(!appendage) {
        return;
    }
    local obj = appendage.getParent();
}

// 额外绘制（如特效叠加）
function drawAppend_appendage_atmage_effect(appendage, isOver, x, y, isFlip)
{
    if(!appendage) {
        return;
    }
    local obj = appendage.getParent();

    if(!obj) {
        appendage.setValid(false);  // 设为false可终止appendage
        return;
    }

    local pAni = sq_GetCurrentAnimation(obj);
    // ... 自定义绘制逻辑 ...
}

// 判断appendage是否应该结束
function isEnd_appendage_atmage_effect(appendage)
{
    if(!appendage) return false;
    local T = appendage.getTimer().Get();
    return false;  // 返回true则appendage结束
}
```

### 可用的appendage钩子函数列表

| 钩子名 | 说明 |
|--------|------|
| `proc` | 每帧调用，核心逻辑 |
| `onStart` | appendage被附加时调用 |
| `onEnd` | appendage被移除时调用 |
| `prepareDraw` | 绘制前调用 |
| `drawAppend` | 额外绘制（特效叠加等） |
| `isEnd` | 判断appendage是否应结束（返回bool） |
| `isDrawAppend` | 判断是否需要额外绘制 |
| `onStartMap` | 进入地图时调用 |
| `onVaildTimeEnd` | 持续时间到期时调用 |
| `onDamageParent` | 宿主受到伤害时调用 |
| `onAttackParent` | 宿主攻击时调用 |
| `onSetHp` | 宿主HP被设置时调用 |
| `onApplyHpDamage` | 宿主受到HP伤害时调用 |
| `getImmuneTypeDamageRate` | 获取免疫类型伤害比率 |
| `onSourceKeyFrameFlag` | 来源对象的关键帧标志触发 |
| `onDestroyObject` | 对象被销毁时调用 |
| `onChangeState` | 宿主状态改变时调用 |

### appendage常用API

```squirrel
// 获取宿主对象
appendage.getParent()

// 获取来源对象（施加appendage的对象）
appendage.getSource()

// 设置appendage有效/无效（false=终止）
appendage.setValid(false)

// 获取计时器
appendage.getTimer().Get()

// 获取关联的技能索引
appendage.sq_GetSkillIndex()

// 添加属性修改
appendage.sq_AddChangeStatus("标识名", obj, obj, 0, CHANGE_STATUS_TYPE_XXX, bool, value)

// 添加光环效果
appendage.sq_AddSquirrelAuraMaster("名称", obj, obj, range, tickInterval, maxCount, something)
```

---

## Header文件常量定义

每个角色的header文件定义了该角色使用的所有常量，供其他脚本引用：

```squirrel
// 状态ID（每个状态有唯一编号，用于pushState注册和状态切换）
STATE_ATTACK              <- 8
STATE_FIRE_ROAD           <- 24
STATE_ICE_SWORD           <- 27

// 技能ID（每个技能有唯一编号）
SKILL_WIND_STRIKE         <- 1
SKILL_FIRE_ROAD           <- 6

// 自定义动画索引（对应技能.skl中定义的动画列表）
CUSTOM_ANI_FIRE_ROAD_CAST1  <- 5
CUSTOM_ANI_FIRE_ROAD_CAST2  <- 6

// 自定义攻击信息索引
CUSTOM_ATTACK_INFO_DARK_CHANGE <- 0

// 通用常量
SKL_STATIC_INT_IDX_0      <- 0    // 技能静态整数索引
SKL_LVL_COLUMN_IDX_0      <- 0    // 技能等级数据列索引
PASSIVEOBJ_SUB_STATE_0    <- 10   // 被动对象子状态

// 元素枚举
ENUM_ELEMENT_FIRE         <- 0
ENUM_ELEMENT_WATER        <- 1
ENUM_ELEMENT_DARK         <- 2
ENUM_ELEMENT_LIGHT        <- 3
ENUM_ELEMENT_NONE         <- 4
```

**注意**：这些常量在header文件中定义后，通过pushScriptFiles加载，之后在该角色的所有脚本中都可以直接使用这些常量名。

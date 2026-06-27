# 86幻境 PVF · 快捷栏装备系统与 Appendage 使用指南

> 本文档基于 `huanjing\sqr`（86幻境客户端已解压脚本，作为权威源）整理，记录三件事：
> 1. **快捷栏装备（勇者纹章）系统的编写方法**
> 2. **一次完整排错过程**（procAppend_Priest 纹章失效 → 定位 → 修复，**根因已实测确认**）
> 3. **sqr 中 Appendage（附加状态）的详细用法**
>
> 所有文件路径均相对 `sqr/` 根目录（DNF 脚本根）。

---

## 目录

- [第一部分：快捷栏装备（勇者纹章）系统](#第一部分快捷栏装备勇者纹章系统)
- [第二部分：排错过程 —— procAppend_Priest 纹章失效](#第二部分排错过程--procappend_priest-纹章失效)
- [第三部分：sqr 中 Appendage 使用详解](#第三部分sqr-中-appendage-使用详解)
- [附录：关键文件索引与命名空间说明](#附录关键文件索引与命名空间说明)

---

# 第一部分：快捷栏装备（勇者纹章）系统

## 1.1 系统概览

"快捷栏装备"是一类放进**快捷栏（消耗品栏）就生效、拿出就失效**的特殊道具，本服表现为 **勇者纹章**（道具代码 `100000101` ~ `100000110`）。

它的效果**不靠道具文件自带的 `[append]` 段**（`.stk` 里没有），而是完全由 sqr 脚本每帧扫描快捷栏状态、动态挂载/卸载对应的 Appendage 来实现。

## 1.2 涉及的文件

| 文件 | 作用 |
|---|---|
| `stackable/crest/1000001xx.stk` | 纹章道具定义（名称、图标、说明）。**无 `[append]` 段**，效果完全交给 sqr |
| `equipment/ap/ap_1000001xx.nut` | 纹章对应的 Appendage 脚本（定义加什么属性） |
| `equipment/equipment_<job>.nut` | 每个职业一个，含 `procAppend_<job>` / `isUsableItem_<job>` / `drawMainCustomUI_<job>` 三个函数 |
| `equipment/equipment_main.nut` | 加载器，`sq_RunScript` 所有 `equipment_<job>.nut` |

> 道具代码与文件：`100000101` = 勇者纹章1（`stackable/crest/100000101.stk`）。

## 1.3 三函数协作机制（核心）

三个函数靠角色身上的一个共享向量 `obj.getVar("itemIndex")`（6 槽）传值：

```
        ┌──────── obj.getVar("itemIndex")  ← 6槽共享向量 ────────┐
        └────────────────────────────────────────────────────────┘
            ▲ 每帧重置                  ▲ 使用时写入                  │ 每帧读取
            │                            │                            │
   ┌────────┴─────────┐         ┌────────┴────────┐         ┌────────┴──────────────┐
   │drawMainCustomUI  │         │ isUsableItem    │         │ procAppend_<job>     │
   │   _<job>         │         │   _<job>        │         │ 扫向量→挂/卸 ap        │
   │ clear + [0..5]   │         │ 记第一个空槽     │         │ sq_AppendAppendage    │
   └──────────────────┘         └─────────────────┘         └──────────────────────┘
                                                                       │
                                                                       ▼
                                                       ┌──────────────────────────────┐
                                                       │ equipment/ap/ap_1000001xx.nut│
                                                       │ onStart → addParameter(+属性)│
                                                       └──────────────────────────────┘
```

### `drawMainCustomUI_<job>(obj)` —— 每帧重置向量

```squirrel
function drawMainCustomUI_Priest(obj) {
    if(!obj) return;
    obj.getVar("itemIndex").clear_vector();
    local slotNum = 6;
    for(local i=0;i<slotNum;++i)
        obj.getVar("itemIndex").push_vector(i);   // 重置成 [0,1,2,3,4,5]（"空槽"标志）
    drawNewDamageUI_AllGrowJob(obj);
}
```
- "空槽"用 `值 == 下标` 来识别（初始 `[0,1,2,3,4,5]`）。一旦写入真实纹章码（如 `100000101`），值就不再等于下标。

### `isUsableItem_<job>(obj, itemIndex)` —— 使用时登记纹章码

引擎在玩家**使用**快捷栏道具时调用（`itemIndex` = 被使用的道具码）。函数把它写进第一个空槽（带去重）：

```squirrel
function isUsableItem_Priest(obj, itemIndex) {
    if (!sq_IsInBattle()) isUsableItem_AllGrowJob(obj, itemIndex);
    local size = obj.getVar("itemIndex").size_vector();
    for (local i = 0; i < size; ++i) {
        local isSaved = false;
        for (local i = 0; i < size; ++i)            // 全表查重（注意内层 local i 遮蔽外层）
            if (obj.getVar("itemIndex").get_vector(i) == itemIndex) isSaved = true;
        local index = obj.getVar("itemIndex").get_vector(i);
        if (index == i && !isSaved)                  // 找到空槽且未登记过 → 写入
            obj.getVar("itemIndex").set_vector(i, itemIndex);
    }
    return duegonUsableItem(itemIndex);
}
```
> ⚠️ 已知瑕疵：内层 `local i` 遮蔽外层（能跑但易碎）；全表扫描冗余 O(n²)。功能上不影响。

### `procAppend_<job>(obj)` —— 每帧扫描并同步 ap

对每个纹章码执行"在栏→挂、离栏→卸"：

```squirrel
// 以 100000101 为例
isAppend1 = CNSquirrelAppendage.sq_IsAppendAppendage(obj, "equipment/ap/ap_100000101.nut");
if(!isAppend1){                       // 还没挂
    local isInQuickSlot = false;
    local size = obj.getVar("itemIndex").size_vector();
    for(local i=0;i<size;++i)
        if(obj.getVar("itemIndex").get_vector(i) == 100000101) isInQuickSlot = true;
    if(isInQuickSlot){
        local appendage = CNSquirrelAppendage.sq_AppendAppendage(obj, obj, -1, false,
                              "equipment/ap/ap_100000101.nut", true);
        CNSquirrelAppendage.sq_Append(appendage, obj, obj);   // 提交激活
    }
}
else if(isAppend1){                   // 已挂
    // 同样扫描；若不在快捷栏 → setValid(false) 摘除
    if(!isInQuickSlot)
        obj.GetSquirrelAppendage("equipment/ap/ap_100000101.nut").setValid(false);
}
```

## 1.4 ap 文件结构（详见第三部分）

`equipment/ap/ap_100000101.nut`（+10 攻防）：

```squirrel
function sq_AddFunctionName(appendage){
    appendage.sq_AddFunctionName("onStart", "onStart_appendage_100000101");
    appendage.sq_AddFunctionName("onEnd",   "onEnd_appendage_100000101");
}

function onStart_appendage_100000101(appendage){
    if(!appendage) return;
    local obj = appendage.getParent();
    if(!obj){ appendage.setValid(false); return; }

    local physicalAttack = 10, physicalDefense = 10;
    local magicalAttack = 10, magicalDefense = 10;
    local stuck = -1;

    local change_appendage = appendage.sq_getChangeStatus("100000101");
    if(!change_appendage)
        change_appendage = appendage.sq_AddChangeStatus("100000101", obj, obj, 0, 37, false, physicalAttack);
    if(change_appendage){
        change_appendage.clearParameter();
        change_appendage.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_ATTACK,  false, physicalAttack.tofloat());
        change_appendage.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_DEFENSE, false, physicalDefense.tofloat());
        change_appendage.addParameter(CHANGE_STATUS_TYPE_MAGICAL_ATTACK,   false, magicalAttack.tofloat());
        change_appendage.addParameter(CHANGE_STATUS_TYPE_MAGICAL_DEFENSE,  false, magicalDefense.tofloat());
        change_appendage.addParameter(CHANGE_STATUS_TYPE_STUCK,            false, stuck.tofloat());
    }
}
```

## 1.5 新增一个快捷栏装备的步骤

1. **写 ap 脚本**：在 `equipment/ap/` 下新建 `ap_<道具码>.nut`，套用上面模板（改 `key`、改属性值）。
2. **写道具文件**：在 `stackable/crest/` 下新建 `<道具码>.stk`，说明里写"请放入快捷栏位置，副本内生效"，`[flavor text]` 写 `快捷栏装备`（无需 `[append]` 段）。
3. **挂扫描块**：在 `equipment/equipment_<job>.nut` 的 `procAppend_<job>` 末尾，照 `100000101` 模板复制一段，把道具码与 ap 路径改成你的。
4. **每个职业都要加**：这套机制是每职业一份 `equipment_<job>.nut`，要让全职业通用需逐个加（或加到公共前缀里）。

## 1.6 已知问题：100000110 段写坏

`equipment_priest.nut` 里 `100000110`（+500 高属性纹章）那一段结构错位，是改代码时的调试残留：

```squirrel
if(isInQuickSlot)
{

}                                              // ← if 体是空的
        local appendage = sq_AppendAppendage(obj, obj, -1, false,
                              "equipment/ap/ap_100000110.nut", true);   // ← 跑到 if 外，每帧无条件挂
//      CNSquirrelAppendage.sq_Append(appendage, obj, obj);            // ← 被注释
...
//      appendage.setValid(false);                                       // ← 摘除也被注释
```
后果：110 不论在不在快捷栏都会被挂上、且摘不掉。修法：照 `100000101` 模板把这段改回去。

---

# 第二部分：排错过程 —— procAppend_Priest 纹章失效

> **结论先行**：根因是 `procAppend_Priest` 前缀里的 `useRangeBuff(obj)` 每帧抛运行时异常（`useRangeBuff_getTeamCount` 内 null 解引用），把函数在纹章代码之前就中断了。**已通过"单独去掉 useRangeBuff 即恢复"实测确认。**

## 2.1 现象

圣职者把勇者纹章放进快捷栏后，**纹章附加的属性不生效**。

## 2.2 排查思路与数据流梳理

按系统化调试法（先找根因，不盲改），逐环节验证"特效要生效必须经过的链路"：

| 环节 | 结论 | 证据 |
|---|---|---|
| ap 文件存在且有效 | ✅ 正常 | `ap_100000101…108、110.nut` 都在；`onStart` 用 `addParameter` 正常加属性 |
| 道具存在 | ✅ 正常 | `100000101` = 勇者纹章1，`stackable/crest/100000101.stk` |
| `.stk` 自带附魔兜底 | ❌ 没有 | `.stk` 无 `[append]` 段，效果 100% 靠 sqr 扫描 |
| 机制在职业间是否一致 | ✅ 一致 | `equipment_swordman.nut` 的纹章段与 priest 逐字节相同 |
| `procAppend_<job>` 是否被调用 | ✅ 是 | 标准引擎钩子，每帧调 |

> 走过的弯路：初步假设曾偏向"`itemIndex` 向量在 procAppend 读取时是空的（写入时机问题）"——**这个假设后来被推翻**。教训：先验证执行流是否跑得到，再怀疑数据。

## 2.3 关键转折

实测发现：把 `procAppend_Priest` 开头的**前缀 5 个调用整段删掉**，纹章就生效了：

```squirrel
// 删掉这 5 行后，纹章恢复
procAppend_AllGrowJob(obj);
setMyCharacterNoShake(obj);
procAppend_DryOut(obj);
useBuffSkills(obj);     // 一键buff
useRangeBuff(obj);      // 群体BUFF
```

这把问题从"纹章代码本身"转向了"前缀里有什么在阻断执行"。随后逐个二分，**最终确认只要去掉 `useRangeBuff(obj)` 一行就恢复**——元凶锁定。

## 2.4 根因（已确认）

**机制：`useRangeBuff` 每帧抛运行时异常，把 `procAppend_Priest` 在纹章代码之前就中断了。**

- Squirrel 异常会被 DNF 引擎在 C++/Squirrel 边界**静默捕获**（不崩溃，但本次函数调用就此中止，后面代码本帧不执行）。
- 纹章的 `sq_AppendAppendage` 位于函数末尾；前缀一抛，末尾整段没跑 → 纹章永不挂载。
- 表现就是"纹章不生效"，且**没有任何报错**。

**抛点：`useRangeBuff` → `useRangeBuff_getTeamCount`**（定义在 `character/new_priest/rangebuff/rangebuff.nut`）：

```squirrel
function useRangeBuff(obj){
    local array = useRangeBuff_getTeamCount(obj);   // ← 每帧无条件执行，在 rangeBuff 开关判断之前
    if(obj.getVar("rangeBuff").getBool(0)){ ... }
}

function useRangeBuff_getTeamCount(obj){
    local array = [];
    local objectManager = obj.getObjectManager();
    for(local i=0; i<objectManager.getCollisionObjectNumber(); i+=1)   // ⚠️① OM 为 null 就抛
    {
        local object = objectManager.getCollisionObject(i);
        if(object && !obj.isEnemy(object) && object.isObjectType(OBJECTTYPE_CHARACTER)){
            local sqrChr = sq_GetCNRDObjectToSQRCharacter(object);
            local isAI = sq_IsAiCharacter(sqrChr);                      // ⚠️② sqrChr 为 null 就抛
            ...
        }
    }
    return array;
}
```

| 抛点 | 触发条件 |
|---|---|
| ① `objectManager.getCollisionObjectNumber()` | `obj.getObjectManager()` 返回 null（角色过渡态/特殊上下文） |
| ② `sq_IsAiCharacter(sqrChr)` | 某个通过类型检查的对象，`sq_GetCNRDObjectToSQRCharacter` 转换返回 null（NPC、召唤物、正在销毁的队友等） |

致命点：`useRangeBuff_getTeamCount` 在 `rangeBuff` 开关**之前**就无条件调用——即使从没用过"群体BUFF"功能，它每帧都在跑，一旦命中 null 就抛。

> 引入路径：`loadstate.nut`(L19) → `Character/priest_load_state.nut`(L129) → `character/newpriest/rangebuff/rangebuff.nut`(L31)。它在 `equipment_main.nut` 之前加载，所以不是"未定义"，而是运行时 null 解引用。

## 2.5 修复方案

**当前采用的修复（方案 A）**：在 `procAppend_Priest` 中**只去掉 `useRangeBuff(obj)` 一行**，保留其余 4 个前缀调用。实测纹章恢复，其它功能（防震屏、干涸、一键buff）正常。代价：失去"群体BUFF（给队友）"功能。

**如需保留群体 BUFF（方案 B）**：把 `rangebuff.nut` 顶部的取队友函数加上 null 保护，再把 `useRangeBuff(obj)` 加回去：

```squirrel
function useRangeBuff_getTeamCount(obj){
    local array = [];
    if(!obj) return array;
    local objectManager = obj.getObjectManager();
    if(!objectManager) return array;                    // 保护①
    local n = objectManager.getCollisionObjectNumber();
    for(local i=0; i<n; i+=1){
        local object = objectManager.getCollisionObject(i);
        if(object && !obj.isEnemy(object) && object.isObjectType(OBJECTTYPE_CHARACTER)){
            local sqrChr = sq_GetCNRDObjectToSQRCharacter(object);
            if(!sqrChr) continue;                        // 保护②
            if(sq_IsAiCharacter(sqrChr)) continue;
            array.push(sq_GetObjectId(object));
        }
    }
    return array;
}
```
改完后前缀 5 行全部保留，纹章与全部功能都正常。

## 2.6 经验教训

1. **"某功能不工作且无报错"** 在 DNF sqr 里常常意味着**脚本抛了被静默吞掉的异常**，导致函数后半段跑不到。优先怀疑"执行流被中断"，而不是"数据没传对"。
2. **定位手法**：在可疑函数里加 `print("[DBG] ...")` 埋点，看每帧执行到哪一行停；或用"逐个加回/删掉前缀调用"的二分法隔离元凶（本次正是后者秒定）。
3. **写每帧函数（procAppend / drawMainCustomUI）时**，任何对象解引用、外部对象遍历都要做 null 保护——一次 null 就会让整个函数本帧失效。
4. **霰弹枪式修复要不得**：一开始"5 个全删"虽然能修好，但会误杀有用功能。隔离出唯一元凶（本次是 `useRangeBuff`）后，只动它一个，功能损失最小。

---

# 第三部分：sqr 中 Appendage 使用详解

## 3.1 Appendage 是什么

Appendage（附加状态 / appendage）是挂载到游戏对象（角色、怪物、被动体）上的**状态效果载体**——BUFF、被动、装备特效、纹章属性等都用它实现。一个 appendage 关联一个 `.nut` 脚本，由 `CNSquirrelAppendage` 统一管理。

特点：
- **独立于角色 state**：挂上就持续生效，与角色技能状态机无关。
- **可设持续时间**：到时自动失效；也可手动 `setValid(false)` 摘除。
- **通过 ChangeStatus 改属性**：在 `onStart` 里用 `addParameter` 修改角色属性。

## 3.2 核心 API（`CNSquirrelAppendage` 命名空间）

| API | 签名 | 作用 |
|---|---|---|
| `sq_AppendAppendage` | `(target, source, skillId, isChangeState, apPath, isAppendStart)` | **创建** appendage；`isAppendStart=true` 则创建即激活 |
| `sq_Append` | `(appendage, parent, source)` | **提交激活**一个用 `isAppendStart=false` 创建的 appendage |
| `sq_AppendAppendageID` | `(appendage, source, target, skillId, isEnable)` | 按技能 ID 提交（可去重/登记来源） |
| `sq_IsAppendAppendage` | `(obj, apPath)` | 查询 obj 身上是否已挂某 ap |
| `sq_RemoveAppendage` | `(obj, apPath)` | 摘除 obj 身上的某 ap |
| `sq_GetAppendage` | `(obj, apPath)` | 取 obj 身上的 appendage 对象 |

以及对象方法：

| 方法 | 作用 |
|---|---|
| `obj.GetSquirrelAppendage(apPath)` | 取 obj 身上的 appendage 对象（同 `sq_GetAppendage`） |
| `appendage.getParent()` | 取被挂载的对象 |
| `appendage.setValid(false)` | 使该 appendage 失效（摘除） |
| `appendage.isValid()` | 是否仍有效 |
| `appendage.sq_SetValidTime(ms)` | 设持续时间（毫秒） |
| `appendage.setAppendCauseSkill(BUFF_CAUSE_SKILL, job, skillId, level)` | 登记来源技能 |
| `appendage.getVar()` / `getVar(name)` | appendage 自带的存储向量，可在 onStart/onEnd 间传值 |

> `apPath` 是相对 `sqr/` 根的路径，如 `"equipment/ap/ap_100000101.nut"`、`"character/atpriest/holylight/ap_holylight.nut"`。

## 3.3 ap 脚本文件结构

每个 ap 脚本必须注册 `onStart`（挂上时）/ `onEnd`（摘除时）回调，并在 `onStart` 里施加效果：

```squirrel
function sq_AddFunctionName(appendage){
    appendage.sq_AddFunctionName("onStart", "onStart_appendage_XXX");
    appendage.sq_AddFunctionName("onEnd",   "onEnd_appendage_XXX");
}

function onStart_appendage_XXX(appendage){
    if(!appendage) return;
    local obj = appendage.getParent();
    if(!obj){ appendage.setValid(false); return; }     // 宿主没了就自删

    local value = 500;
    local change = appendage.sq_getChangeStatus("XXX");   // 先看有没有
    if(!change)
        change = appendage.sq_AddChangeStatus("XXX", obj, obj, 0, 37, false, value);  // 没有就建
    if(change){
        change.clearParameter();
        change.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_ATTACK, false, value.tofloat());
        // ... 更多 addParameter
    }
}

function onEnd_appendage_XXX(appendage){
    if(!appendage) return;
    // 清理：例如关闭光谱特效
    local spectrum = appendage.sq_GetOcularSpectrum("ocularSpectrum");
    if(spectrum) spectrum.endCreateSpectrum();
}
```

### `sq_AddChangeStatus` / `addParameter` 参数说明

```squirrel
change = appendage.sq_AddChangeStatus(key, sourceObj, targetObj, 0, 37, false, baseValue);
```
- `key`：字符串标识（一般用道具码/技能码，如 `"100000101"`）。
- `sourceObj` / `targetObj`：来源与目标。
- 第 4、5 位：内部类型码（照搬 `0, 37` 即可）。
- `false`：是否百分比标志（一般 false）。
- `baseValue`：基准值（会再被 `addParameter` 的具体值覆盖）。

```squirrel
change.addParameter(CHANGE_STATUS_TYPE_XXX, isPercent, value.tofloat());
```
- `CHANGE_STATUS_TYPE_XXX`：要改的属性类型（见 3.5）。
- `isPercent`：该值是绝对值(false)还是百分比(true)。
- `value`：数值（推荐 `.tofloat()`）。

## 3.4 两种挂载模式

### 模式 A：即时挂载（创建即激活）

适合"不需要提前配置"的简单场景：

```squirrel
CNSquirrelAppendage.sq_AppendAppendage(obj, obj, -1, false, "xxx/ap.nut", true);
//                                                                   ↑ isAppendStart=true
```

### 模式 B：延迟挂载（创建 → 配置 → 提交）

适合"挂上之前要先设持续时间 / 来源技能"的场景（如觉醒、爆发态）：

```squirrel
// ① 创建但不激活（末参 false）
local ap = CNSquirrelAppendage.sq_AppendAppendage(obj, obj, SKILL_X, false, "xxx/ap.nut", false);

// ② 配置
local change_time = sq_GetLevelData(obj, SKILL_X, 0, skill_level);
ap.sq_SetValidTime(change_time);                                          // 持续时长
ap.setAppendCauseSkill(BUFF_CAUSE_SKILL, sq_getJob(obj), SKILL_X, skill_level);  // 来源

// ③ 提交激活
CNSquirrelAppendage.sq_Append(ap, obj, obj);
// 或按技能ID提交：CNSquirrelAppendage.sq_AppendAppendageID(ap, obj, obj, SKILL_X, true);
```

> 实例见 `character/common/burster/burster.nut:280-292`。

## 3.5 常用 `CHANGE_STATUS_TYPE_*` 类型

取自 `ap_100000110.nut`（高属性纹章，加了一整套）：

| 常量 | 含义 |
|---|---|
| `CHANGE_STATUS_TYPE_PHYSICAL_ATTACK` | 物理攻击力 |
| `CHANGE_STATUS_TYPE_MAGICAL_ATTACK` | 魔法攻击力 |
| `CHANGE_STATUS_TYPE_PHYSICAL_DEFENSE` | 物理防御力 |
| `CHANGE_STATUS_TYPE_MAGICAL_DEFENSE` | 魔法防御力 |
| `CHANGE_STATUS_TYPE_EQUIPMENT_PHYSICAL_ATTACK` | 装备物理攻击力 |
| `CHANGE_STATUS_TYPE_EQUIPMENT_MAGICAL_ATTACK` | 装备魔法攻击力 |
| `CHANGE_STATUS_TYPE_ADDITIONAL_PHYSICAL_GENUINE_ATTACK` | 额外物理独立攻击 |
| `CHANGE_STATUS_TYPE_ADDITIONAL_MAGICAL_GENUINE_ATTACK` | 额外魔法独立攻击 |
| `CHANGE_STATUS_TYPE_ELEMENT_ATTACK_ALL` | 全属性强化 |
| `CHANGE_STATUS_TYPE_PHYSICAL_CRITICAL_DAMAGE_RATE` | 物理暴击伤害率 |
| `CHANGE_STATUS_TYPE_MAGICAL_CRITICAL_DAMAGE_RATE` | 魔法暴击伤害率 |
| `CHANGE_STATUS_TYPE_STUCK` | 命中（负值=减命中） |

## 3.6 完整示例：多属性纹章

`equipment/ap/ap_100000110.nut`（+500 全属性）：

```squirrel
function sq_AddFunctionName(appendage){
    appendage.sq_AddFunctionName("onStart", "onStart_appendage_100000110");
    appendage.sq_AddFunctionName("onEnd",   "onEnd_appendage_100000110");
}

function onStart_appendage_100000110(appendage){
    if(!appendage) return;
    local obj = appendage.getParent();
    if(!obj){ appendage.setValid(false); return; }

    local physicalAttack=500, physicalDefense=500, magicalAttack=500, magicalDefense=500;
    local equipmentphysicalAttack=500, equipmentmagicalAttack=500;
    local equipmentphysicalgenuine=500, equipmentmagicalgenuine=500;
    local attackall=500, physicalrate=500, magicaltate=500;

    local change = appendage.sq_getChangeStatus("100000110");
    if(!change)
        change = appendage.sq_AddChangeStatus("100000110", obj, obj, 0, 37, false, physicalAttack);
    if(change){
        change.clearParameter();
        change.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_ATTACK, false, physicalAttack.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_DEFENSE, false, physicalDefense.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_MAGICAL_ATTACK, false, magicalAttack.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_MAGICAL_DEFENSE, false, magicalDefense.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_EQUIPMENT_PHYSICAL_ATTACK, false, equipmentphysicalAttack.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_EQUIPMENT_MAGICAL_ATTACK, false, equipmentmagicalAttack.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_ADDITIONAL_PHYSICAL_GENUINE_ATTACK, false, equipmentphysicalgenuine.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_ADDITIONAL_MAGICAL_GENUINE_ATTACK, false, equipmentmagicalgenuine.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_ELEMENT_ATTACK_ALL, false, attackall.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_CRITICAL_DAMAGE_RATE, false, physicalrate.tofloat());
        change.addParameter(CHANGE_STATUS_TYPE_MAGICAL_CRITICAL_DAMAGE_RATE, false, magicaltate.tofloat());
    }
}
```

## 3.7 常见陷阱

1. **`isAppendStart` 参数别搞反**：`true` = 创建即激活（无需再 `sq_Append`）；`false` = 仅创建，需 `sq_Append` 提交。延迟模式下忘调 `sq_Append` 会挂不上。
2. **`onStart` 里必须保护 `getParent()`**：宿主可能为 null，直接 `setValid(false)` 自删，否则后续解引用会抛。
3. **持续时间**：不调 `sq_SetValidTime` 的 appendage 可能瞬时失效；需要持久的效果要么不设时长（看引擎默认），要么显式设一个大值，或由外部 `setValid(false)` 控制（如纹章"在栏才挂、离栏才卸"）。
4. **ap 路径相对 `sqr/`**：写错路径会挂不上（且可能静默失败）。
5. **挂 ap 的宿主函数不能抛异常**：`sq_AppendAppendage` 通常写在 `procAppend_<job>` 这类每帧函数里；如果它之前的代码抛异常，挂载代码就跑不到（本次排错的根因正是这点）。
6. **去重**：同一 ap 重复挂载前先用 `sq_IsAppendAppendage` 判断，避免叠挂。

---

# 附录：关键文件索引与命名空间说明

## A.1 本服的模组命名空间

86幻境是叠加了多个模组的客户端，脚本里有几个明显的"作者命名空间"：

| 命名空间 | 含义 |
|---|---|
| `js60_qq506807329/` | 模组作者 js60（QQ 506807329），大量 `share_obj/` 共享被动体与各职业扩展 |
| `apjh/` | 公共钩子层（`procAppend_AllGrowJob`、`isUsableItem_AllGrowJob`、`duegonUsableItem`、`drawNewDamageUI_AllGrowJob` 等） |
| `qq358332886_monster_appendage/` | 怪物 appendage |
| `unclebang_shared_passive_object/` | 共享被动体 |
| `custom/content/` | 自定义内容（训练房、新技能键、新段位系统等） |

## A.2 快捷栏装备相关函数索引

| 函数 | 文件 | 行 |
|---|---|---|
| `procAppend_Priest` | `equipment/equipment_priest.nut` | 38 |
| `isUsableItem_Priest` | `equipment/equipment_priest.nut` | 2 |
| `drawMainCustomUI_Priest` | `equipment/equipment_priest.nut` | 24 |
| `procAppend_AllGrowJob` | `apjh/procappend.nut` | 1 |
| `isUsableItem_AllGrowJob` | `apjh/lue.nut` | 39 |
| `duegonUsableItem` | `apjh/lue.nut` | 58 |
| `drawNewDamageUI_AllGrowJob` | `apjh/drawmaincustomui.nut` | 1 |
| `setMyCharacterNoShake` | `character/common/common_common.nut` | 638 |
| `procAppend_DryOut` | `character/new_priest/priest_common.nut` | 696 |
| `useBuffSkills` | `character/new_priest/releasebuffs/releasebuffs.nut` | 41 |
| `useRangeBuff`（**本次元凶**） | `character/new_priest/rangebuff/rangebuff.nut` | 31 |

## A.3 脚本加载入口

`sqr/loadstate.nut` 是入口，关键加载顺序：

```
loadstate.nut
  L7   dnf_enum_header.nut
  L8   common.nut                          (含 setState_AllGrowJob、IsInterval 等)
  L10  Character/common_load_state.nut     (common_header / common_common / burster)
  L14  js60_qq506807329/js60_506807329_common.nut
  L19  Character/priest_load_state.nut
         L129 → character/newpriest/rangebuff/rangebuff.nut   (useRangeBuff)
  L31  apjh/lue.nut                        (AllGrowJob 系列钩子)
  L43  equipment/equipment_main.nut        (各 equipment_<job>.nut，含 procAppend_<job>)
  L47  common_class.nut / common_function.nut / ui 等
```

> 注意：`Character/<job>_load_state.nut` 都在 `equipment_main.nut`（L43）**之前**加载，所以前缀里调用的 `procAppend_AllGrowJob`、`useRangeBuff` 等在 `procAppend_<job>` 运行时都已全局可见——本例的 bug 不是"未定义"，而是运行时 null 解引用。

---

*文档生成自对 `huanjing\sqr` 实际源码的逐文件分析；排错结论已通过实测确认（单独去掉 `useRangeBuff` 即恢复）。如需复核某段，按附录索引定位对应文件即可。*

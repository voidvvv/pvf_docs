---
title: 新增一件快捷栏装备（纹章）
category: howto
tags: [快捷栏, 纹章, appendage, 实操]
sources: ["快捷栏装备系统与Appendage使用指南.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 新增一件快捷栏装备（纹章）

机制背景见 [[quickbar-equipment-system]]；ap 语法见 [[appendage]]。

## 四步流程

### 1. 写 ap 脚本

`equipment/ap/ap_<道具码>.nut`，套模板（改 key 与属性值）：

```squirrel
function sq_AddFunctionName(appendage){
    appendage.sq_AddFunctionName("onStart", "onStart_appendage_<道具码>");
    appendage.sq_AddFunctionName("onEnd",   "onEnd_appendage_<道具码>");
}
function onStart_appendage_<道具码>(appendage){
    if(!appendage) return;
    local obj = appendage.getParent();
    if(!obj){ appendage.setValid(false); return; }
    local physicalAttack = 10;   // …按需改属性与数值
    local change = appendage.sq_getChangeStatus("<道具码>");
    if(!change)
        change = appendage.sq_AddChangeStatus("<道具码>", obj, obj, 0, 37, false, physicalAttack);
    if(change){
        change.clearParameter();
        change.addParameter(CHANGE_STATUS_TYPE_PHYSICAL_ATTACK, false, physicalAttack.tofloat());
        // 更多 addParameter 见 [[appendage]] 常量表；多属性范例：ap_100000110.nut
    }
}
```

### 2. 写道具文件

`stackable/crest/<道具码>.stk`：说明写"请放入快捷栏位置，副本内生效"，`[flavor text]` 写 `快捷栏装备`，**无需 `[append]` 段**。

### 3. 挂扫描块

在 `equipment/equipment_<job>.nut` 的 `procAppend_<job>` 末尾，照 `100000101` 模板复制一段：

```squirrel
local isAppendX = CNSquirrelAppendage.sq_IsAppendAppendage(obj, "equipment/ap/ap_<道具码>.nut");
if(!isAppendX){
    local isInQuickSlot = false;
    local size = obj.getVar("itemIndex").size_vector();
    for(local i=0;i<size;++i)
        if(obj.getVar("itemIndex").get_vector(i) == <道具码>) isInQuickSlot = true;
    if(isInQuickSlot){
        local appendage = CNSquirrelAppendage.sq_AppendAppendage(obj, obj, -1, false,
                              "equipment/ap/ap_<道具码>.nut", true);
        CNSquirrelAppendage.sq_Append(appendage, obj, obj);
    }
} else {
    if(!isInQuickSlot)   // 注意：else 分支需同样扫描
        obj.GetSquirrelAppendage("equipment/ap/ap_<道具码>.nut").setValid(false);
}
```

### 4. 每个职业都要加

机制是每职业一份 `equipment_<job>.nut`，全职业通用需逐个加（或加到公共前缀 `procAppend_AllGrowJob`，位于 `apjh/procappend.nut`）。

## 完成后

`import_file` 写回全部改动文件 → `save_as_pvf` 重新打包（见 [[pvfutility-mcp]]）。

## ⚠️ 前置检查（血泪教训）

- 确认目标职业的 `procAppend_<job>` **前缀调用没有会抛异常的函数**（每帧异常会静默阻断后面的挂载代码，见 [[procappend-priest-bug]]）；
- 对照 `100000110` 的写坏案例，确认 if/else 结构完整、`sq_AppendAppendage` 没有跑到条件外；
- `.stk` 道具码须与扫描块、ap 路径三方一致。

## 相关

- 道具族：[[crest-items]]　·　来源摘要：[[src-quickbar-guide]]

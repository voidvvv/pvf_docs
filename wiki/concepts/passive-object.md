---
title: 被动物体（Passive Object）系统
category: concept
tags: [passiveobject, po, obj, 弹道]
sources: ["input/my_inclusion.md", "AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 被动物体（Passive Object）系统

被动对象（po）是独立于角色的游戏对象：弹道、特效区域、召唤物等。**三件套结构**：

1. **`.obj` 文件**（`passiveobject/` 目录）—— 定义视觉与碰撞属性；
2. **`.nut` 脚本**（`sqr/` 下，`po_` 前缀）—— 定义行为逻辑；
3. **`passiveobject.lst` 注册** —— 数字 ID ↔ `.obj` 路径映射（格式：`ID -> passiveobject/…/xxx.obj "名称"`，如 `24212`）。

## 注册与创建链路

```
passiveobject.lst 注册 .obj ID
        ↓
load_state: IRDSQRCharacter.pushPassiveObj("…/po_xxx.nut", ID)   ← 见 [[load-state]]
        ↓
技能脚本关键帧: obj.sq_SendCreatePassiveObjectPacket(ID, 子状态, x, y, z)
```

## 技能 → 被动对象的数据传递

技能侧写入自定义数据（按序），po 侧 `setCustomData` 按同样顺序读取：

```squirrel
// 技能侧
obj.sq_StartWrite();
obj.sq_WriteWord(pauseTime);   obj.sq_WriteDword(damage1);
obj.sq_WriteByte(maxHit);
obj.sq_SendCreatePassiveObjectPacket(passiveObjectIndex, 0, x, y, z);
// 或按位置：sq_SendCreatePassiveObjectPacketPos(mage, idx, 0, x, y, z);

// po 侧
function setCustomData_po_XXX(obj, receiveData){
    local pauseTime = receiveData.readWord();
    local damage1   = receiveData.readDword();
    local maxHit    = receiveData.readByte();
}
```

## po 脚本钩子

| 函数 | 时机 |
|------|------|
| `setCustomData_po_{名称}(obj, receiveData)` | 创建时读取技能传入数据 |
| `procAppend_po_{名称}(obj)` | 每帧 |
| `onAttack_po_{名称}(obj, damager, boundingBox, isStuck)` | 命中目标 |
| `onEndCurrentAni_po_{名称}(obj)` | 动画播完（常用于销毁） |
| `onKeyFrameFlag_po_{名称}(obj)` | 关键帧 |

销毁：`onEndCurrentAni` 中 `if(obj.isMyControlObject()) sq_SendDestroyPacketPassiveObject(obj);`

## .obj 文件格式示例

```
[floating height]  1                          // 浮空高度
[layer]            `[bottom]`                 // 渲染层级
[pass type]        `[pass all]`               // 穿透类型
[piercing power]   1000                       // 穿透力
[basic motion]     `Animation/ATFireRoad/ATFireRoad1.ani`
[attack info]      `AttackInfo/ATFireRoad1.atk`
[etc attack info]  `AttackInfo/ATFireRoad2.atk`   [/etc attack info]  // 多段攻击判定
[name]             `火焰爆炸`
```

## 实测注意事项（input/raw 结论）

- **自定义 po 无法指定生成方式**：创建包的"生成方式"数字只能用默认 `0`（正常生成）；最后一个字符串参数代表 ptl（详见 `input/raw/passiveobject_notice.txt`，索引：[[input-raw-references]]）。
- 攻击类型字段（`[attack type]`、`[ignore defense]` 等）示例见 `input/raw/passiveobject_attacktype.txt`、`passiveobject_attack_test.txt`（注意后者为 UTF-16 编码）。

## 相关

- 注册机制：[[load-state]]　·　技能脚本侧：[[nut-scripting]]　·　技能分析：[[analyze-skill]]
- 来源摘要：[[src-my-inclusion]]

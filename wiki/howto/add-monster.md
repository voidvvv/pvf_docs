---
title: 流程：新增一只怪物
category: howto
tags: [howto, 怪物, 新增, 模板]
sources: []
created: 2026-11-06
updated: 2026-11-06
status: current
---

# 流程：新增一只怪物

> 需求形态："给我加一个会 XX 机制的怪 / 复制某怪改成 XX"。先读 [[monster-dir]]/[[monster-act]]/[[monster-ai]] 背景再动手。

## 0. 选模板（最重要的一步）

**不做从零发明**：挑一个机制最接近的既有怪整套复制改（推荐：`newmonsters/powerstation/fuhrman/` 全套样例、`110levelnormaldungeon/vericollis/boss/michael/` 机制 Boss、`newmonsters/metrocenter/apc/knight/` APC）。新式怪目录结构：`<怪名>.mob + action/ + attackinfo/ + ai/ + animation/`。

## 1. 写 .mob

必备段（参考模板怪逐段抄）：`[name]`（中文直写）、`[level]`、`[category]`、`[ability category]`（六维倍率）、关键抗性、`[weight]/[hit recovery]`、动作槽（waiting/move/sit/damage1/2/down/overturn + `[attack action]` 列表）、`[attack info]`（**与 attack action 逐行对齐，无判定动作留空行**）、`[cooltime]`（与上两表对齐；**或**旧式 `[attack kind]`，二选一）、`[ai pattern]` 四件套、`[warlike]/[think term]/[vision]`。

## 2. 动作三件套

- 每个攻击动作写 `action/xx.act`：`[MOTION]`（引 animation/ 的 .ani）+ `[TRIGGER]/[BEHAVIOR]` 帧时间轴（指令查 [[monster-behavior-commands]]）；
- 有判定的动作写 `attackinfo/xx.atk`（字段见 [[monster-atk]]）；
- 纯动画/纯位移动作可以不写 atk（attack info 留空行）。

## 3. AI

四件套至少写 Action.ai（条件树 → `return 动作索引`，语法见 [[monster-ai]]）；Event/DestinationSelect/MoveMethod 可恒值（`return -1/0/0`）。简单怪可省 [ai pattern] 用 [intelligence]+[attack kind]。

## 4. 注册 monster.lst

- 条目：`新ID\r\n\`相对路径.mob\`` 追加到 `monster/monster.lst`（两行、ID 在前、路径不带 `monster/` 前缀）；
- **新 ID 选号**：先 GET 封包实况 lst 扫空闲段（勿按行号/顺延猜，ID 稀疏；参考既有段位——6 万段=新怪、20 万段=活动）；
- 动画等资源直接复用既有 .ani（不复制也能引相对路径）。

## 5. 摆进房间（可选：只注册不摆=词典可用）

目标地下城的 .map 里加 `[monster]` 行：`新ID 等级规则 x y z 1 1 [fixed] [normal]`（字段表见 [[dungeon-chain]]）；或新开房间/替换空位见 [[modify-dungeon-map]]。

## 6. 导入打包

1. 新文件放 `huanjing/monster/...`（保持目录结构）；
2. `import_files_batch`（大批量走直连 HTTP API，见 [[pvfutility-mcp]]）；
3. `save_as_pvf` —— ⚠️ 目标=当前载入文件时会写成 `Script(1).pvf`，**关 pvfUtility 后手动换名**；
4. 游戏内验证：词典查怪、进图打一场、确认每招表现。

## 检查单

- [ ] 三表（attack action/attack info/cooltime）行数一致
- [ ] .act 引用的 .ani/.ptl 相对路径存在（大小写与磁盘一致）
- [ ] CREATE PASSIVEOBJECT / SUMMON MONSTER 的 ID 已在对应 lst 注册（passiveobject 查封包）
- [ ] monster.lst 新条目格式=两行、ID 不与既有冲突
- [ ] [TRIGGER] n [ON/OFF] 与 [DO BEHAVIOR][ME] n 的序号未越界（全文件 0 基！）
- [ ] 已 save_as_pvf 并替换游戏 PVF

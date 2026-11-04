---
title: 参考资料索引：input/raw
category: entity
tags: [参考, nut, passiveobject]
sources: ["input/raw/nut_internal_functions.txt", "input/raw/nut_note.txt", "input/raw/passiveobject_notice.txt"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 参考资料索引：input/raw

`input/raw/`（源工作区 `D:\pvf` 内）存放 5 个原始参考 txt。**按需查阅原文，本页记录各文件内容与查阅时机。**

| 文件 | 内容 | 查阅时机 | 注意 |
|------|------|----------|------|
| `nut_internal_functions.txt` | nut 系统**内置钩子函数**全表（系统自动调用的函数及触发时机） | 写新脚本前查"有哪些钩子可用" | 注释为中文 |
| `nut_note.txt` | 注册原语笔记（`pushScriptFiles` / `pushState` 等）+ **职业枚举** `ENUM_CHARACTERJOB_*`（0=SWORDMAN、1=FIGHTER…） | 需要 `pushState` 职业参数时 | 部分韩文注释乱码 |
| `passiveobject_notice.txt` | **实测结论**：自定义 po 无法指定生成方式，只能用默认 `0`；创建参数最后字符串代表 ptl | 写自定义被动对象时 | — |
| `passiveobject_attacktype.txt` | po 攻击类型字段示例（`[attack type]`、`[ignore defense]`、`[magic]` 等） | 配 .obj / .atk 攻击属性 | — |
| `passiveobject_attack_test.txt` | po 伤害演示字段（`[damage bonus]`、`[weapon damage apply]` 等） | 调伤害参数 | **UTF-16 编码**，直接 Read 会见宽字符间隔 |

## 消化状态

- 核心结论（po 生成方式 = 0、职业枚举、钩子概念）已并入 [[passive-object]]、[[load-state]]、[[nut-scripting]]；
- 函数全表未全量搬运——遇到"这个钩子存在吗 / 什么时候触发"时打开 `nut_internal_functions.txt` 原文，值得沉淀的条目按 [[WIKI]] §5.1 回填。

## 相关

- 教程库：[[pvfdoc-library]]　·　入门文档：[[src-my-inclusion]]

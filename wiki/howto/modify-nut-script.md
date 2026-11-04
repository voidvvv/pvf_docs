---
title: 修改 / 新增 .nut 脚本流程
category: howto
tags: [nut, 实操, sqr]
sources: ["AGENTS.md", "input/my_inclusion.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 修改 / 新增 .nut 脚本流程

语法与钩子约定见 [[nut-scripting]]；注册机制见 [[load-state]]。

## 通用步骤

```
1. 定位：search_pvf / get_file_list（或本地 huanjing/sqr/ 检索）
2. 读原文：get_file_content 确认结构与钩子命名约定
3. 修改 / 新增脚本（Squirrel 语法；遵循既有命名空间与 ap_/po_ 前缀）
4. 新增对象时在对应 .lst 注册 ID（appendage.lst / passiveobject.lst / skill lst…）
5. 新增脚本 / 状态 / 被动对象时挂到对应 load_state（pushScriptFiles / pushState / pushPassiveObj）
6. import_file 写回 → save_as_pvf 重新打包
```

## 修改检查单

- [ ] 钩子函数命名与注册名一致（`sq_AddFunctionName` 的第二参数 = 实际函数名字符串）；
- [ ] 每帧函数（proc / procAppend / drawMainCustomUI）内解引用均有 null 保护（见 [[procappend-priest-bug]]）；
- [ ] 新增 ap / po 的 ID 与 `.lst`、load_state 三方一致；
- [ ] 常量在对应 header 定义后再引用；
- [ ] 未改动原版文件名拼写（见 [[naming-traps]]）；
- [ ] 已重新打包 PVF 并替换游戏文件。

## 常见任务入口

| 任务 | 参考页 |
|------|--------|
| 新增快捷栏装备 / 纹章 | [[add-quickbar-equipment]] |
| 掉率等数据表（.etc）修改 | [[modify-drop-rate]] |
| 技能（.skl）分析 | [[analyze-skill]] |
| appendage API 用法 | [[appendage]] |
| 被动物物创建链路 | [[passive-object]] |

## 相关

- 工具：[[pvfutility-mcp]]　·　来源摘要：[[src-agents-md]]、[[src-my-inclusion]]

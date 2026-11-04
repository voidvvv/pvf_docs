---
title: 工具：frida.js 运行时 Hook
category: entity
tags: [工具, frida, hook]
sources: ["AGENTS.md", "frida.js"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 工具：frida.js 运行时 Hook

源工作区根目录的 `frida.js` 是 **Frida 运行时注入脚本**，用于 hook 服务端 / 客户端函数——已知的注入目标包括：绝望之塔、怪物攻城、时装潜能等游戏逻辑。

## 与 PVF 修改的互补关系

| 途径 | 改什么 | 生效方式 |
|------|--------|----------|
| **PVF 修改**（本 wiki 主体） | 静态数据 / 脚本（`.etc` / `.nut` / `.skl`…） | 重新打包 `Script.pvf`，游戏加载时生效 |
| **frida.js**（非 PVF 路径） | 运行时函数行为 | 注入即生效，**不改动 PVF** |

选择标准：能通过 PVF 数据 / 脚本实现的，优先走 PVF（可持久、随封包分发）；PVF 覆盖不到的引擎层逻辑，才用 Frida。

## 使用要点

- 配合 Frida 注入服务端 / 客户端进程运行（`frida -f <目标> -l frida.js` 类方式）；
- 具体 hook 点与函数签名以源工作区 `frida.js`（源工作区 `D:\pvf` 内）当前内容为准；
- 本页为工具索引页；hook 实例积累后应拆分为独立 debugging / howto 页并回填此处链接。

## 相关

- 静态修改工具：[[pvfutility-mcp]]
- 来源摘要：[[src-agents-md]]

---
title: NPC 交易价格：[price] 与 [value]
category: concept
tags: [price, value, npc, 经济]
sources: []
created: 2026-11-04
updated: 2026-11-04
status: current
---

# NPC 交易价格：[price] 与 [value]

装备与物品（`.equ` / `.stk`）都有与 NPC 交易的价格字段：

| 标签 | 含义 |
|------|------|
| `[price]` | 从 NPC 处**买入**的价格 |
| `[value]` | 卖给 NPC 的价格**基础值**——**实际卖出价 = `[value]` ÷ 5** |

### 换算示例

- `[value]` = 10000 → 卖给 NPC 只得 **2000**；
- 实测样本 `equipment/character/common/ring/100322605.equ`：`[price]` 120000、`[value]` 96000 → 卖出价 19200（印证两字段并存；÷5 规则本身以下方来源为准）。

### 修改提示

- 想让某装备/物品卖店更值钱：调 `[value]`（记住玩家到手是 ÷5 后的数），而非 `[price]`；
- `[price]` 影响商店回购/购买侧（如 NPC 贩卖列表，参考 [[pvfdoc-library]]【06】NPC-商店）；
- 改完照常 `import_file` + `save_as_pvf`（见 [[pvfutility-mcp]]）。

## 来源与置信

- **来源：用户实测告知（2026-11-04 会话）**——÷5 规则与两标签语义均为用户确认的一手结论，当前无 pvfDoc 文档佐证；后续若在 pvfDoc 找到书面记载，在此追加引用。
- 字段样本印证：`100322605.equ`（见 [[equ-format]]）。

## 相关

- 字段所在文件：[[equ-format]]（.equ）、[[src-gift-box]]（.stk 侧）
- 礼盒/道具价格相关模板：[[pvfdoc-library]]【16】礼包类模板

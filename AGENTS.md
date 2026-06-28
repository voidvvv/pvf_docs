# AGENTS.md — DNF 幻境版 PVF 工作区

> 本文件面向在此仓库中工作的 AI 编程助手，提供项目背景、目录约定、工具用法与避坑要点。
> 写给 AI 看的"项目说明书"，请先通读再动手。

---

## 一、项目是什么

这是一个 **DNF（地下城与勇士）"86 幻境"（huanjing）版本的 PVF 模组 / 分析工作区**。

- **PVF**（`Script.pvf`）是 DNF 的数据封包格式，游戏客户端与服务端共同读取。
- 工作目标：**解析、修改、扩展 DNF 的数据表（`.etc`）与脚本（`.nut`），再编译回 PVF 供游戏加载**。
- 当前已载入的 PVF 源（用 `mcp__pvfutility__get_pvf_pack_file_path` 可查）：
  `C:\myFiles\happy\danji\DNF\86幻境客户端\86幻境客户端\Script.pvf`

---

## 二、必读约定（最容易踩坑，务必先看）

1. **分清两套数据源**：
   - `huanjing/`（仓库内）= **本地工作副本**（已解包的 PVF 内容），可用 Read/Edit 直接改，**已被 gitignore，不入库**。
   - `pvfutility` MCP 操作的是**已载入的 PVF 封包**（路径见上），是游戏真正读取的内容。
   - 读权威内容 → 用 MCP；本地增量编辑 → 改 `huanjing/` 或用 MCP `import_file` 写回；**最终都必须 `save_as_pvf` 重新打包才生效**。

2. **真实 DNF 路径 = 去掉 `huanjing/` 前缀**。
   例：`huanjing/etc/worlddrop.etc` → DNF 内路径 `etc/worlddrop.etc`。**MCP 调用传 DNF 路径（不带 `huanjing/`）**。

3. **所有概率类数值以 1,000,000 为基数**：实际值 = 数值 ÷ 1,000,000。
   例：`1000` = 千分之一 = 0.1%（**不是必掉**）；`1,000,000` 才是 100%。`[basis of rarity dicision]`、`worlddrop.etc` 等均如此。

4. **原版文件名拼写不要"纠正"**：如 `itemdropinfo_monseter.etc`（`monseter` 是原版 typo），改名会导致游戏无法识别。

5. **`.etc` 表用 Tab 分隔、`[section]` 分段**，修改时保留原有空白与缩进。

6. **`.nut` 是 Squirrel 脚本**，通过钩子函数扩展游戏；改完同样要重新打包 PVF。

7. **不要把 `Script.pvf` / `huanjing/` 提交进 git**（已在 `.gitignore`）。仓库只跟踪：文档、`skills/`、`frida.js`、`input/`、`pvfDoc/`。

---

## 三、目录结构

```
/                         # 项目根
├── AGENTS.md                   # 本文件
├── Script.pvf                  # PVF 封包（gitignore）
├── huanjing/                   # 「幻境」版本 DNF 文件根 = DNF 游戏根（gitignore）
│   ├── etc/                    # DNF etc 目录：数据表（167 .etc / 6 .lst / 4 .tbl / …）
│   ├── sqr/                    # DNF sqr 目录：Squirrel 脚本（.nut）
│   └── sqr copy/               # sqr 的备份
├── pvfDoc/pvfDoc/              # PVF 修改教程库（按主题分 19 类，见 §9）
├── skills/
│   └── dnf-skill-analyzer/     # 技能分析 skill（SKILL.md）
├── input/
│   ├── my_inclusion.md         # nut / PVF 结构入门（强烈建议先读）
│   └── raw/                    # 参考资料 txt（nut 内部函数、passiveobject 说明等）
├── frida.js                    # Frida 运行时 hook 脚本（服务端逻辑注入）
└── *.md                        # 既有分析文档（见 §8）
```

### `huanjing/etc/`（数据表，共 200 项）

DNF 的纯数据 / 配置。代表性文件：

| 文件 | 作用 |
|------|------|
| `etc/itemdropinfo_monster_hell.etc` | 深渊怪物掉率 |
| `etc/itemdropinfo_monseter.etc` | 普通怪物掉率 |
| `etc/itemdropinfo_monseter_extra.etc` | 精英怪物掉率 |
| `etc/worlddrop.etc` | 世界掉落（按 1~200 等级分段，`-1` 分隔） |
| `etc/*.lst` | ID → 文件路径 映射索引（equipment / stackable / skill / passiveobject 等） |

### `huanjing/sqr/`（脚本，.nut）

按职能 / 角色分目录：

| 子目录 | 文件数 | 作用 |
|--------|-------|------|
| `sqr/character/` | ~1768 | 角色脚本（按职业：swordman/fighter/gunner/mage/priest/thief 及 at* 转职） |
| `sqr/map/` | 64 | 地图脚本 |
| `sqr/monster/` | 62 | 怪物脚本 |
| `sqr/qq358332886_monster_appendage/` | 65 | 第三方怪物 appendage |
| `sqr/js60_qq506807329/` | 58 | 第三方公共脚本 |
| `sqr/ui/` | 32 | UI 脚本 |
| `sqr/equipment/` | 25 | 装备逻辑（快捷栏装备、纹章等） |
| `sqr/passiveobject/` | 17 | 被动物体脚本 |
| `sqr/appendage/` `apjh/` `custom/` `functions/` `unclebang_shared_passive_object/` | — | appendage / 自定义 / 公共函数 / 共享被动物体 |

`sqr/` 顶层还有全局脚本：`common.nut`、`common_function.nut`、`dnf_enum_header.nut`、`init_character.nut`、`loadstate.nut` 等。

---

## 四、PVF 文件格式速览

| 后缀 | 类型 | 位置 | 说明 |
|------|------|------|------|
| `.etc` / `.tbl` | 数据表 | `etc/` | `[section]` 分段、Tab 分隔的文本表 |
| `.nut` | Squirrel 脚本 | `sqr/` | 钩子函数扩展游戏逻辑 |
| `.skl` | 技能定义 | `skill/<职业>/` | 技能属性、等级数据；解析见 `skills/dnf-skill-analyzer` |
| `.apd` | appendage 定义 | `appendage/` | 声明式状态效果（change status / dummy 等） |
| `.obj` | 被动物体定义 | `passiveobject/` | 视觉与碰撞属性，配合同名 `.nut` |
| `.lst` | 索引表 | 各目录 | 数字 ID → 文件路径 / 名称 映射 |

### 核心概念

- **appendage（ap）**：挂在角色 / 怪物上的"状态"。两种定义方式：
  1. `appendage/*.apd` + 在 `appendage.lst` 注册 ID（可被装备 `[appendage]` 标签引用）；
  2. `sqr/**/*.nut` 中用 `sq_AddFunctionName` 注册钩子（`proc` / `onStart` / `onEnd` / …）。
- **被动对象（passive object）**：弹道 / 特效 / 召唤物等独立对象 = `.obj`（视觉）+ `.nut`（行为）；在 `passiveobject.lst` 注册 ID，技能里用 `sq_SendCreatePassiveObjectPacket(ID,…)` 创建。
- **load_state 文件**：`{职业}_load_state.nut` 是脚本注册入口（`pushScriptFiles` / `pushState` / `pushPassiveObj`）。
- 详细入门见 [`input/my_inclusion.md`](input/my_inclusion.md)。

---

## 五、核心工具：pvfutility MCP

读取 / 修改 PVF 内容的**首选工具**（比手写文件解析可靠）：

| 工具 | 用途 |
|------|------|
| `get_pvf_pack_file_path` | 查询当前载入的 PVF 路径 |
| `get_pvf_root_directory` | 列出 PVF 根目录 |
| `get_file_list` / `get_all_lst_file_list` | 列目录 / 列所有 LST |
| `get_file_content` | 读单个文件（可指定编码 TW/CN/KR/JP/UTF8） |
| `get_file_contents_batch` | 批量读 |
| `search_pvf` | 按关键词 / 正则搜索（可限定目录） |
| `get_lst_file_info` | 读 LST 索引 |
| `item_code_to_file_info` / `get_item_info` | 物品代码 → 文件 / 物品名 |
| `import_file` / `import_files_batch` | 写回 / 覆盖文件内容到 PVF |
| `save_as_pvf` | **另存为新 PVF 封包（重新打包）** |

> 路径参数传 **DNF 内路径**（不带 `huanjing/`），如 `etc/worlddrop.etc`、`sqr/equipment/equipment_swordman.nut`。

---

## 六、常见任务流程

### A. 读 / 分析某文件
1. `search_pvf` 或 `get_file_list` 定位 → 2. `get_file_content` 读取 → 3. 按 §4 格式解析，数值按 §2.3 换算。

### B. 修改数据表（.etc）
1. `get_file_content` 读原文 → 2. 编辑（Tab 分段、`[section]` 结构不变）→ 3. `import_file` 写回 → 4. `save_as_pvf` 重新打包。
（也可直接 Edit `huanjing/etc/` 本地副本，但务必同步到 PVF 并打包。）

### C. 修改脚本（.nut）
- 遵循 Squirrel 语法与钩子函数命名约定（见 `input/my_inclusion.md`）；
- 新增 appendage / 被动对象记得在对应 `.lst` 注册 ID；
- 改完重新打包。

### D. 分析 / 修改技能
- 直接用 skill：`skills/dnf-skill-analyzer`（已封装 `.skl` 解析、`[level property]` 解码流程）。

### E. 运行时 hook（非 PVF 路径）
- `frida.js` 用 Frida 注入服务端 / 客户端函数（绝望之塔、怪物攻城、时装潜能等），与 PVF 修改互补。

---

## 七、概率与数值约定（重要）

- **基数 1,000,000**：掉率 / 稀有度判定 / 世界掉落 等概率字段，实际值 = 数值 ÷ 1,000,000。
- **稀有度判定**（`[basis of rarity dicision]`）：累计阈值，各档爆率 = (本档阈值 − 上档阈值) ÷ 1,000,000；末列为难度码（如 1000001 / 1000002），不参与计算。
- **世界掉落**（`worlddrop.etc`）：`<等级> 0` + 若干 `<物品ID> <掉率>` 对，等级块用 `-1` 分隔；每物品独立判定。
- 详见 [`物品掉率文件修改说明.md`](物品掉率文件修改说明.md)。

---

## 八、既有分析文档（可复用，勿重复造轮子）

| 文档 | 内容 |
|------|------|
| [`物品掉率文件修改说明.md`](物品掉率文件修改说明.md) | 三个怪物掉率表 + worlddrop 的完整结构与修改方法 |
| [`快捷栏装备系统与Appendage使用指南.md`](快捷栏装备系统与Appendage使用指南.md) | 快捷栏装备编写、appendage 用法、排错记录 |
| [`gift_box_analysis.md`](gift_box_analysis.md) | 礼盒分析 |
| [`atswordman_render_analysis.md`](atswordman_render_analysis.md) | 女鬼剑渲染分析 |
| [`input/my_inclusion.md`](input/my_inclusion.md) | nut / PVF / appendage / 被动对象 入门 |

---

## 九、参考资料库

`pvfDoc/pvfDoc/` 下按主题分类的教程（JSON），按需查阅：

| 目录 | 主题 |
|------|------|
| `【01】PVF文件解读` | PVF 格式基础 |
| `【02】装备修改` … `【14】宠物修改` | 各类对象修改教程 |
| `【16】常用模板【复制可用】` | 可复制模板 |
| `【18】掉落` | 掉落系统 |
| `【04】技能修改` | 技能修改（含中英对照、全职业对照表） |

---

## 十、工作风格备忘

- 修改前先 `get_file_content` 读原文，确认结构再改；**绝不臆造字段含义**，拿不准就查 `pvfDoc/` 或 `input/my_inclusion.md`。
- 概率 / 数值改动需复算并标注（基数 1,000,000）。
- 改完提醒用户「需 `save_as_pvf` 重新打包并替换游戏 PVF 后才生效」。
- 文档 / 注释统一用中文，与既有文档风格一致。

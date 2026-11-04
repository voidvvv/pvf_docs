# D:\pvf → LLM Wiki 知识库改造方案

> 本文档是**改造方案**，尚未对项目做任何改造。依据 [`llm_wiki.md`](llm_wiki.md) 的模式（Karpathy LLM Wiki），针对本仓库（DNF 86幻境 PVF 工作区）的具体情况设计。
> 你确认本方案后，我再按 §9 的阶段逐步执行。
>
> **三个硬性需求的满足方式**（先给结论）：
> | 需求 | 满足方式 |
> |------|----------|
> | ① 在别的会话中让 AI 大模型引用知识库 | wiki 为**纯 Markdown + 自包含 Schema**，零工具依赖；任何 LLM（Claude Code / Codex / GPT 等）拿到 `wiki/` 目录、读 `WIKI.md` 即可查询与维护（§8） |
> | ② 支持增加新知识 | Schema 中定义 **Ingest 协议**：新文档 / 会话结论 / 口述经验 三种入口（§7.1） |
> | ③ 支持修改已有知识 | Schema 中定义 **Update 协议**：修正、矛盾标注、过时归档 三种处理（§7.2） |

---

## 一、现状盘点：知识源清单（Raw Sources 层）

`llm_wiki.md` 三层架构中的 **Raw sources（不可变原始资料）**，在本项目中对应以下内容。**这些文件改造后一律不动**，wiki 页面通过 frontmatter `sources:` 字段指回它们：

| # | 来源 | 类型 | 知识价值 | 初始处理方式 |
|---|------|------|----------|--------------|
| 1 | `物品掉率文件修改说明.md` | 分析文档 | 掉率三件套 + worlddrop 完整结构、概率基数约定 | **全量 ingest**（提炼出多个概念页/实体页/流程页） |
| 2 | `快捷栏装备系统与Appendage使用指南.md` | 分析文档 | 快捷栏装备机制、Appendage 用法、排错实录 | **全量 ingest** |
| 3 | `input/my_inclusion.md` | 入门文档 | nut/PVF 结构、load_state 机制 | **全量 ingest** |
| 4 | `gift_box_analysis.md` | 分析文档 | 礼盒系统分析 | ingest 为摘要页 + 按内容拆页 |
| 5 | `atswordman_render_analysis.md` | 分析文档 | 女鬼剑渲染分析 | ingest 为摘要页 + 按内容拆页 |
| 6 | `AGENTS.md` | 项目说明书 | 工具用法、目录映射、全部核心约定 | **全量 ingest**（它是目前最好的"准 schema"） |
| 7 | `input/raw/*.txt`（5 个） | 原始参考 | nut 内部函数表、passiveobject 说明 | 建索引页 + 按需 ingest（体积大，不预提炼） |
| 8 | `pvfDoc/pvfDoc/`（19 类 JSON 教程） | 外部教程库 | 各类 PVF 修改教程 | 只建**导航索引页**，按需逐类 ingest |
| 9 | `skills/dnf-skill-analyzer/SKILL.md` | 技能定义 | .skl 解析流程 | 摘要页 + 指针 |
| 10 | `huanjing/`、`Script.pvf`（游戏数据本体） | **数据源** | 非文档知识 | **不进 wiki**（见下方"边界声明"） |

### 边界声明（重要架构决策）

- **进 wiki 的**：关于数据的知识 —— 文件结构、规律、修改方法、排错经验、工具用法。
- **不进 wiki 的**：数据本身 —— `huanjing/` 下 200 个 `.etc`、数千个 `.nut` 的具体内容。这些通过 pvfutility MCP 实时查询（那是"原始资料中的原始资料"，wiki 页面只教"怎么查、怎么改"）。

---

## 二、目标架构：三层映射

完全对应 `llm_wiki.md` 的 Architecture 一节：

```
┌─────────────────────────────────────────────────────────────┐
│  Raw sources（不可变）                                        │
│  = 上表 #1~#9：既有 *.md / input/ / pvfDoc/ + 游戏数据(MCP查)  │
│  你和 LLM 只读，永不修改                                       │
├─────────────────────────────────────────────────────────────┤
│  Wiki 层（LLM 全权写作维护）★新建★                             │
│  = wiki/ 目录，约 25~30 个初始 Markdown 页                     │
│  概念页 / 实体页 / 流程页 / 排错页 / 来源摘要页 / 索引 / 日志     │
├─────────────────────────────────────────────────────────────┤
│  Schema 层（人与 LLM 共同演化）★新建★                          │
│  = wiki/WIKI.md —— 自包含的"维护者手册"                        │
│  任何 LLM 读完它就成为一名守规矩的 wiki 维护者                   │
└─────────────────────────────────────────────────────────────┘
```

### 目标目录树

```
wiki/                          ← 新目录，git 跟踪（可整体复制/clone 到任何地方）
├── WIKI.md                    ← Schema：结构约定 + 操作协议（自包含，见 §5 草案）
├── index.md                   ← 内容目录：全页面清单 + 一句话摘要（每次 ingest 更新）
├── log.md                     ← 操作日志：append-only，格式见 §6.3
├── overview.md                ← 全局综述：知识地图 + 当前综合结论（synthesis）
├── concepts/                  ← 概念页（跨文件复用的领域知识）
├── entities/                  ← 实体页（具体文件 / 目录 / 工具 / 道具 / 系统）
├── howto/                     ← 操作流程页（step-by-step）
├── debugging/                 ← 排错案例页（现象→定位→根因→修复）
└── sources/                   ← 来源摘要页（每份原始文档一页，指向 raw 原文）
```

> ⚠️ **不使用 OMC 自带的 `.omc/wiki/`**：它默认 gitignore、项目本地、依赖 OMC 插件，无法满足"在别的地方被任何 AI 引用"的需求。本方案的 `wiki/` 是独立、开放、可携带的权威知识库（两者关系详见 §10）。

---

## 三、初始页面规划（Bootstrap 清单）

首次批量 ingest 产出的页面。文件名用英文 slug（利于 `[[链接]]`、命令行、跨平台），页内标题用中文。

### concepts/（概念页，8 页）

| slug | 标题 | 主要来源 |
|------|------|----------|
| `pvf-format` | PVF 封包与文件类型体系（.etc/.nut/.skl/.apd/.obj/.tbl/.lst） | AGENTS.md §4、pvfDoc【01】 |
| `nut-scripting` | nut（Squirrel）脚本体系：目录结构、文件类型、命名规则 | my_inclusion.md |
| `load-state` | load_state 注册机制（pushScriptFiles / pushState / pushPassiveObj） | my_inclusion.md |
| `appendage` | Appendage：两种定义方式（.apd + lst 注册 / nut 钩子） | 快捷栏指南第三部分 |
| `passive-object` | 被动物体：.obj + .nut + passiveobject.lst 三件套 | AGENTS.md、input/raw/passiveobject_*.txt |
| `probability-basis` | 概率基数 1,000,000 约定与稀有度累计阈值算法 | 物品掉率文档、AGENTS.md §7 |
| `etc-table-format` | .etc 表格式规范（[section] 分段、Tab 分隔、缩进保持） | AGENTS.md §2 |
| `huanjing-path-mapping` | 目录映射：huanjing/ 前缀 ⇄ DNF 内路径 | 物品掉率文档·前置章 |

### entities/（实体页，8 页）

| slug | 标题 | 主要来源 |
|------|------|----------|
| `worlddrop-etc` | etc/worlddrop.etc：世界掉落结构（等级段、-1 分隔、独立判定） | 物品掉率文档·第九章 |
| `itemdropinfo-family` | 掉率三件套：monseter / monseter_extra / monster_hell 差异 | 物品掉率文档 |
| `quickbar-equipment-system` | 快捷栏装备系统：三函数协作 + itemIndex 共享向量机制 | 快捷栏指南·第一部分 |
| `crest-items` | 勇者纹章道具族（100000101~110） | 快捷栏指南 |
| `pvfutility-mcp` | 工具：pvfutility MCP 全接口用法与路径约定 | AGENTS.md §5 |
| `frida-js` | 工具：frida.js 运行时 hook（与 PVF 修改的互补关系） | AGENTS.md、frida.js |
| `pvfdoc-library` | 参考库导航：pvfDoc 19 类教程目录 + 各类一句话简介 | pvfDoc/ |
| `input-raw-references` | 参考资料索引：input/raw 五个 txt 的内容与查阅时机 | input/raw/ |

### howto/（流程页，4 页）

| slug | 标题 | 主要来源 |
|------|------|----------|
| `modify-drop-rate` | 修改怪物/世界掉率的完整流程（读→改→import→save_as_pvf） | 物品掉率文档、AGENTS.md §6.B |
| `add-quickbar-equipment` | 新增一件快捷栏装备（纹章）的完整步骤 | 快捷栏指南 |
| `modify-nut-script` | 修改/新增 .nut 脚本与 LST 注册流程 | AGENTS.md §6.C、my_inclusion.md |
| `analyze-skill` | 用 dnf-skill-analyzer 解析 .skl 的流程 | skills/dnf-skill-analyzer |

### debugging/（排错页，2 页）

| slug | 标题 | 主要来源 |
|------|------|----------|
| `procappend-priest-bug` | 案例：procAppend_Priest 纹章失效（现象→定位→根因→修复） | 快捷栏指南·第二部分 |
| `naming-traps` | 陷阱集：原版拼写（monseter）、改名即失效类问题 | AGENTS.md §2.4 |

### sources/（来源摘要页，6 页）

每份核心原始文档一页：`src-drop-rate-doc`、`src-quickbar-guide`、`src-my-inclusion`、`src-gift-box`、`src-atswordman-render`、`src-agents-md`。内容 = 3~5 句核心结论 + 原文相对路径 + "本文档被哪些 wiki 页引用"。

### 根级特殊页（3 页）

`index.md`（目录）、`log.md`（日志）、`overview.md`（综述——全部 ingest 完成后最后写，作为知识地图与跨页综合结论）。

**初始规模合计：约 31 页。**

---

## 四、Schema 设计：`wiki/WIKI.md` 内容草案

这是整个改造的核心交付物——它自包含，任何 LLM 读了就能接手维护。草案全文如下（执行阶段 0 时按此落地，之后由你与 LLM 共同演化）：

```markdown
# WIKI.md — 本知识库的 Schema（维护者手册）

> 任何 LLM agent（Claude Code / Codex / 其他）在读写本 wiki 之前，必须先通读本文件并遵循其中的协议。

## 1. 这是什么

这是一个关于 **DNF（地下城与勇士）86幻境版本 PVF 模组开发** 的知识库，由 LLM 写作和维护。
它沉淀"关于数据的知识"（结构、规律、方法、排错），不存放数据本身（游戏文件内容用
pvfutility MCP 实时查询，本 wiki 只教怎么查、怎么改）。

## 2. 三层结构

- **Raw sources（不可变）**：位于源工作区 D:\pvf 的既有文档（根目录 *.md、input/、
  pvfDoc/）。只读。每页 frontmatter 的 sources 字段指回它们。
- **Wiki（本目录，LLM 全权维护）**：你在的地方。你创建、更新、交叉链接这些页面。
- **Schema（本文件）**：约定与协议。修改本文件需人类确认。

## 3. 目录与命名

| 目录 | 用途 |
|------|------|
| concepts/ | 概念页：跨文件复用的领域知识 |
| entities/ | 实体页：具体文件/目录/工具/道具/系统 |
| howto/ | 操作流程页 |
| debugging/ | 排错案例页 |
| sources/ | 原始文档摘要页 |
| （根） | WIKI.md / index.md / log.md / overview.md |

- 文件名 = 英文 kebab-case slug；页内首行 `# 中文标题`。
- 单页目标 50~150 行；超过 200 行应拆分。
- 交叉引用用 `[[slug]]`（Obsidian 兼容）。首次提及概念时链接，每页至少有 1 个入链或出链。

## 4. 页面格式

每个页面必须带 YAML frontmatter：

    ---
    title: 中文标题
    category: concept   # concept|entity|howto|debugging|source|meta
    tags: [掉率, etc]    # 小写，中文可
    sources: ["物品掉率文件修改说明.md"]  # 指向 raw 原文（相对源工作区路径）
    created: YYYY-MM-DD
    updated: YYYY-MM-DD
    status: current      # current|stale|disputed
    ---

正文结构建议：一句话定义 → 详细内容（表格优先）→ 与其他页面的关系（[[链接]]）→
（可选）已知争议/待验证事项。

## 5. 操作协议

### 5.1 Ingest（吸收新知识）—— 用户给你新文档/新结论时
1. 通读来源，与用户确认要点（批量时可从简）。
2. 写 sources/ 摘要页（若是新文档）。
3. 把知识点拆解到对应 concepts/ entities/ howto/ debugging/ 页：
   - 已有页 → 更新该页，刷新 updated，必要时改 status；
   - 没有页 → 新建。
4. 检查受影响页面的交叉引用；更新 index.md。
5. 向 log.md 追加一条记录（§6 格式）。
一次 ingest 允许触碰 5~15 个文件，这是正常且被鼓励的。

### 5.2 Query（查询）—— 用户提问时
1. 先读 index.md 定位相关页 → 读页面 → 综合作答，引用页面名。
2. 若综合过程产生了有价值的新分析（对比表、结论、关联发现），
   **回填为新的 wiki 页**并在 log 记录 —— 好答案不应消失在聊天记录里。
3. 发现知识缺口 → 告知用户，建议补充来源或新建待验证页。

### 5.3 Update（修改已有知识）—— 见 §7 矛盾处理规则
- 修正笔误/过时：直接改，updated 刷新，log 记录原因。
- 新旧信息冲突：**不要删除旧结论**，在页内加"⚠️ 矛盾记录"小节并列两种说法与来源，
  双方 status 改 disputed，等人类裁决。裁决后更新并在 log 记录裁决结果。
- 整页过时：status 改 stale，内容移入页底"历史记录"节，index 中标注。

### 5.4 Lint（健康检查）—— 用户要求或每 ~20 次 ingest 后
检查并修复：页面间矛盾 / 无入链的孤儿页 / 被频繁提及却没有自己页面的概念 /
失效的 [[链接]] / 超长页面 / index 与实际页面不一致。结果写入 log。

## 6. log.md 格式（append-only）

    ## [YYYY-MM-DD] ingest | 来源或主题
    - 新增：concepts/xxx（一句话）
    - 更新：entities/yyy（改了什么）
    - 备注：任何矛盾或待办

每条以 `## [日期] 操作 | 主题` 开头，保证 `grep "^## \[" log.md` 可解析。
禁止修改或删除历史条目。

## 7. 路径约定（可移植性）

- 本 wiki 可能被复制到其他机器/其他 agent 使用。涉及源工作区的路径一律写成
  相对路径并注明"（源工作区 D:\pvf 内）"。
- 游戏内路径（etc/…、sqr/…）按 DNF 约定书写，不带 huanjing/ 前缀的含义
  见 concepts/huanjing-path-mapping。
```

---

## 五、页面模板示例（落地时的实际样子）

以 `concepts/probability-basis.md` 为例：

```markdown
---
title: 概率基数 1,000,000 约定
category: concept
tags: [掉率, 概率, etc]
sources: ["物品掉率文件修改说明.md", "AGENTS.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# 概率基数 1,000,000 约定

所有掉率 / 稀有度判定类数值以 **1,000,000 = 100%** 为基数，实际值 = 数值 ÷ 1,000,000。

| 数值 | 实际含义 |
|------|----------|
| 1000 | 0.1%（千分之一，**不是必掉**） |
| 10000 | 1% |
| 1000000 | 100% |

## 稀有度判定算法
游戏 roll 一个 [0, 1000000) 随机数，按 [basis of rarity dicision] 累计阈值划档；
各档爆率 = (本档阈值 − 上档阈值) ÷ 1,000,000；末列难度码不参与计算。

## 适用范围
[[worlddrop-etc]]、[[itemdropinfo-family]]、[basis of rarity dicision] 段等。

## 相关
- 来源摘要：[[src-drop-rate-doc]]
- 实际修改步骤：[[modify-drop-rate]]
```

---

## 六、index.md 与 log.md 格式

### index.md（内容目录，每次 ingest 更新）

```markdown
# 知识库目录
> 共 31 页 · 最后更新 2026-11-04 · 维护协议见 [[WIKI]]

## concepts/
- [概率基数约定](concepts/probability-basis) — 1,000,000=100%，掉率换算与稀有度算法
- [Appendage](concepts/appendage) — 附加状态的两种定义方式与钩子函数
…

## entities/
…
```

### log.md（append-only 时间线）

```markdown
## [2026-11-XX] bootstrap | 初始批量 ingest
- 新增：31 页（concepts×8 / entities×8 / howto×4 / debugging×2 / sources×6 / meta×3）
- 来源：物品掉率文件修改说明.md、快捷栏装备系统与Appendage使用指南.md、…
```

---

## 七、增量维护协议（对应需求②③）

### 7.1 增加新知识的三种入口

| 入口 | 场景 | 动作 |
|------|------|------|
| **A. 新原始文档** | 你写了/找到了新分析文档、教程 | 完整 Ingest（§5.1 五步：读→摘要页→拆解到概念/实体页→index→log） |
| **B. 会话结论** | 某次会话中排除了 bug、发现了规律 | 轻量 Ingest：直接写入/并入相关页面 + log 一条 `## [日期] ingest \| 会话结论：主题` |
| **C. 口述经验** | 你直接说"记住：xxx" | 同 B，由 LLM 判断并入已有页还是新建页 |

### 7.2 修改已有知识的三种情形

| 情形 | 动作 | 痕迹 |
|------|------|------|
| 笔误/表述优化 | 直接编辑页面 | `updated` 刷新 + log 一条 |
| 新信息与旧页矛盾 | 双说并列（"⚠️ 矛盾记录"小节），`status: disputed`，等人类裁决 | log 记录矛盾与裁决 |
| 结论整体过时 | `status: stale`，正文移入"历史记录"节，index 标注 | log 记录 |

### 7.3 防漂移机制

- 每页 `sources:` 指回 raw 原文 → 任何时候可核对原始出处；
- `status` 字段 + lint 协议（§5.4）周期性清理矛盾与孤儿页；
- git 版本历史兜底（所有增改都有 commit 记录）。

---

## 八、跨会话使用指南（对应需求①）

wiki 是**纯 Markdown + 自包含 Schema**，不依赖任何特定工具。四种典型用法：

| 场景 | 做法 |
|------|------|
| **1. 本机其他 Claude Code 会话** | 新会话中 `cd wiki 目录` 或用 `--add-dir D:\pvf\wiki`；开场提示词：*"请先读 WIKI.md 并遵循其协议，然后回答：……"* |
| **2. 其他 AI 工具（Codex / GPT / Cursor 等）** | 把 `wiki/` 整个目录（含 WIKI.md）作为附件/工作目录给它。WIKI.md 是写给**任何** LLM 的，不绑定 Claude |
| **3. 其他电脑 / 远程协作** | git 管理：`wiki/` 随本仓库提交，clone 即得；也可后续拆成独立仓库或推 GitHub 私有仓（§11 决策点 1） |
| **4. 低 token 快速引用** | 只投喂 `index.md + overview.md`（约 100 行）即可让远端模型知道"库里有什么"，需要时再取具体页面 |

> 可选增强（当前 31 页规模**不需要**，>100 页后再考虑）：[qmd](https://github.com/tobi/qmd) 本地搜索引擎（BM25+向量，CLI/MCP 双形态）；Obsidian 打开 wiki/ 目录获得图谱视图与 `[[链接]]` 跳转（链接语法已兼容）。

---

## 九、实施步骤（你批准后执行）

| 阶段 | 内容 | 产出 | 预计 |
|------|------|------|------|
| **0. 骨架** | 建 `wiki/` 目录树；落地 WIKI.md（§4 草案）；初始化空的 index.md / log.md | 可用的空知识库 | 一次会话内 |
| **1. Bootstrap ingest ①** | ingest `物品掉率文件修改说明.md` + `AGENTS.md` → 约 12 页 | 掉率/工具/路径/概念页 | 同上或下次 |
| **2. Bootstrap ingest ②** | ingest `快捷栏装备指南` + `my_inclusion.md` → 约 12 页 | appendage/nut/快捷栏/排错页 | 1 次会话 |
| **3. Bootstrap ingest ③** | ingest `gift_box_analysis.md`、`atswordman_render_analysis.md`、pvfDoc 导航、input/raw 索引 → 约 7 页 | 摘要页与索引页 | 1 次会话 |
| **4. 综述 + Lint** | 写 overview.md（知识地图）；跑首次 lint（孤儿页/断链/矛盾）；修 index | 健康的初始库 | 1 次会话 |
| **5. 挂钩** | AGENTS.md 增加一节"wiki 知识库"指向 wiki/WIKI.md（让本项目未来会话自动知晓）；git 提交 | 融入项目 | 随阶段 4 |

每阶段结束我会给你看 index/log 摘要，你确认后进入下一阶段（符合 llm_wiki.md "逐份 ingest、保持参与"的建议）。

---

## 十、与现有机制的关系

| 机制 | 关系 |
|------|------|
| **OMC wiki 工具**（`.omc/wiki/`，MCP 的 wiki_* 接口） | 不作为目标库（gitignore + 插件绑定 + 项目本地）。本项目会话中可继续用它做临时记录，但**权威知识库 = `wiki/` 目录** |
| **AGENTS.md / CLAUDE.md** | 保持"项目说明书"角色（面向 PVF 工作 itself）；阶段 5 加一节互链。知识沉淀职能移交 wiki |
| **git** | `wiki/` 入库获得版本历史（注意确认未被 .gitignore 误伤——目前只忽略了 huanjing/ 等，`wiki/` 会被正常跟踪） |
| **pvfutility MCP** | wiki 页面引用它作为数据查询通道，二者互补：wiki=方法论，MCP=实时数据 |

---

## 十一、待你确认的决策点

1. **仓库形态**：`wiki/` 放本仓库内 git 跟踪（推荐，简单）？还是独立 git 仓库 / 推 GitHub 便于多机同步？
2. **初始范围**：按 §3 全量 31 页（推荐）？还是先只做掉率+Appendage 两条线试点 ~15 页？
3. **文件名语言**：英文 slug + 中文标题（推荐，互操作性好）？还是中文文件名（与根目录现有文档一致）？
4. **pvfDoc 处理**：先只建导航页、按需 ingest（推荐，19 类全量太大）？还是指定某几类优先 ingest（如【04】技能、【18】掉落）？

---

*方案完。依据：[llm_wiki.md](llm_wiki.md)（Karpathy LLM Wiki 模式）× 本仓库现状盘点。确认或调整 §11 后即可开始阶段 0。*

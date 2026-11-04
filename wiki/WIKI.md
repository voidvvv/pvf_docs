---
title: WIKI.md — Schema（维护者手册）
category: meta
tags: [schema, 协议]
sources: ["llm_wiki.md", "wiki改造方案.md"]
created: 2026-11-04
updated: 2026-11-04
status: current
---

# WIKI.md — 本知识库的 Schema（维护者手册）

> 任何 LLM agent（Claude Code / Codex / 其他）在读写本 wiki 之前，**必须先通读本文件并遵循其中的协议**。
> 本文件即 [llm_wiki 模式](../llm_wiki.md) 中的 schema 层；修改本文件需人类确认。

## 1. 这是什么

这是一个关于 **DNF（地下城与勇士）86幻境版本 PVF 模组开发** 的知识库，由 LLM 写作和维护。

它沉淀"**关于数据的知识**"——文件结构、规律、修改方法、排错经验、工具用法。
它**不存放数据本身**——游戏文件的具体内容（`huanjing/` 下 200 个 `.etc`、数千个 `.nut`）应通过 pvfutility MCP 实时查询，本 wiki 只教"怎么查、怎么改"。

## 2. 三层结构

| 层 | 位置 | 权限 |
|----|------|------|
| **Raw sources（原始资料）** | 源工作区 `D:\pvf` 的既有文档：根目录 `*.md`、`input/`、`pvfDoc/` | 只读，永不修改 |
| **Wiki（本目录）** | 你在的地方 | LLM 全权写作与维护 |
| **Schema（本文件）** | `wiki/WIKI.md` | 人与 LLM 共同演化，改动需人类确认 |

每个 wiki 页面通过 frontmatter 的 `sources:` 字段指回 raw 原文（路径相对源工作区 `D:\pvf`）。

## 3. 目录与命名

| 目录 | 用途 |
|------|------|
| `concepts/` | 概念页：跨文件复用的领域知识（格式约定、机制、体系） |
| `entities/` | 实体页：具体文件 / 目录 / 工具 / 道具 / 系统 |
| `howto/` | 操作流程页（step-by-step） |
| `debugging/` | 排错案例页（现象 → 定位 → 根因 → 修复） |
| `sources/` | 原始文档摘要页（每份核心文档一页，指向 raw 原文） |
| （根） | `WIKI.md` / `index.md` / `log.md` / `overview.md` |

约定：

- 文件名 = **英文 kebab-case slug**；页内首行 `# 中文标题`。
- 单页目标 50~150 行；超过 200 行应拆分。
- 交叉引用用 `[[slug]]`（Obsidian 兼容）。首次提及概念时链接；每页至少有 1 个入链或出链（根级文件除外）。

## 4. 页面格式

每个页面必须带 YAML frontmatter：

```yaml
---
title: 中文标题
category: concept      # concept | entity | howto | debugging | source | meta
tags: [掉率, etc]       # 小写，中文可用
sources: ["物品掉率文件修改说明.md"]   # 指向 raw 原文（相对源工作区路径），可多个
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: current        # current | stale | disputed
---
```

正文结构建议：一句话定义 → 详细内容（**表格优先于长文**）→ 与其他页面的关系（`[[链接]]`）→（可选）已知争议 / 待验证事项。

## 5. 操作协议

### 5.1 Ingest（吸收新知识）——用户给你新文档 / 新结论时

1. 通读来源，与用户确认要点（批量时可从简）。
2. 若是新文档 → 写 `sources/` 摘要页。
3. 把知识点拆解到对应的 `concepts/` `entities/` `howto/` `debugging/` 页：
   - 已有页 → 更新该页，刷新 `updated`，必要时改 `status`；
   - 没有页 → 新建。
4. 检查受影响页面的交叉引用；更新 `index.md`。
5. 向 `log.md` 追加一条记录（§6 格式）。

一次 ingest 允许触碰 5~15 个文件，这是正常且被鼓励的。

### 5.2 Query（查询）——用户提问时

1. 先读 `index.md` 定位相关页 → 读页面 → 综合作答，**引用页面名**（如"据 [[probability-basis]]"）。
2. 若综合过程产生了有价值的新分析（对比表、结论、关联发现），**回填为新的 wiki 页**并在 log 记录——好答案不应消失在聊天记录里。
3. 发现知识缺口 → 告知用户，建议补充来源或新建"待验证"页。

### 5.3 Update（修改已有知识）

- **修正笔误 / 表述优化**：直接改，`updated` 刷新，log 记录原因。
- **新旧信息冲突**：**不要删除旧结论**。在页内加"⚠️ 矛盾记录"小节，并列两种说法与来源；相关页 `status` 改 `disputed`，等人类裁决。裁决后更新页面并在 log 记录裁决结果。
- **整页过时**：`status` 改 `stale`，正文移入页底"历史记录"节，`index.md` 中标注。

### 5.4 Lint（健康检查）——用户要求时，或每 ~20 次 ingest 后

检查并修复：

- 页面间的事实矛盾；
- 无入链的孤儿页（根级文件与 sources/ 除外）；
- 被多处频繁提及、却没有自己页面的概念；
- 失效的 `[[链接]]`（指向不存在的 slug）；
- 超过 200 行的页面（拆分）；
- `index.md` 与实际页面清单不一致；
- frontmatter 缺字段。

结果写入 `log.md`（`lint` 类型条目）。

## 6. log.md 格式（append-only）

```markdown
## [YYYY-MM-DD] 操作类型 | 来源或主题
- 新增：concepts/xxx（一句话）
- 更新：entities/yyy（改了什么）
- 备注：矛盾 / 待办 / 裁决
```

- 操作类型：`ingest` / `query` / `update` / `lint` / `bootstrap`。
- 每条以 `## [日期] 操作 | 主题` 开头，保证 `grep "^## \[" log.md` 可解析最近操作。
- **禁止修改或删除历史条目**；新条目追加在文件末尾。

## 7. 路径与可移植性约定

- 本 wiki 可能被复制到其他机器 / 交给其他 agent 使用。涉及源工作区的路径一律写**相对路径**并注明"（源工作区 `D:\pvf` 内）"。
- 游戏内路径（`etc/…`、`sqr/…`）按 DNF 约定书写；`huanjing/` 前缀的映射关系见 [[huanjing-path-mapping]]。
- raw 原文不在 wiki 目录内时（通常如此），`sources:` 中仍写相对工作区路径——wiki 页面本身必须**自包含**（复制走之后知识完整，raw 只在需要核对原文时用）。

## 8. 跨会话使用（给携带本 wiki 的人）

- **同机 Claude Code**：新会话 `cd` 到 wiki 目录（或 `--add-dir`），提示词："请先读 WIKI.md 并遵循其协议，然后……"。
- **其他 AI 工具**：把整个 `wiki/` 目录（含本文件）作为附件 / 工作目录；本手册不绑定任何特定 LLM。
- **快速引用**：只投喂 `index.md + overview.md` 即可让远端模型知道"库里有什么"，需要时再取具体页面。
- 详细用法见源工作区 `wiki改造方案.md` §8。

## 9. 领域红线（继承自项目 AGENTS.md，对知识页同样适用）

- 概率类数值一律标注基数 1,000,000（见 [[probability-basis]]）。
- 原版文件名拼写（如 `itemdropinfo_monseter.etc`）**不能"纠正"**，文档中引用时保持原样并提示（见 [[naming-traps]]）。
- 修改 PVF 的任何结论都要提醒：需 `save_as_pvf` 重新打包并替换游戏 PVF 后才生效。

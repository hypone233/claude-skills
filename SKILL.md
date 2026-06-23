---
name: claude-memory
description: "Layered project memory system for persistent AI context across sessions. Designed to live inside an Obsidian vault — memory files and project knowledge interlink via [[wikilinks]]. Use when starting a project, recording decisions, tracking progress, documenting environment quirks, or resuming after a break. Generates 6 structured markdown files with a lightweight _index.md (50-80 lines) for fast context restoration — no more re-explaining project background every session."
---

# Claude Memory — Obsidian-Native Project Memory

## 核心哲学：为什么必须配合 Obsidian

AI 编程助手的最大浪费不是 token 消耗，是**上下文重建**。每次新会话，你要重新解释项目背景、环境配置、之前做到了哪。一次 5 分钟，一年上百次。

简单的办法是让 AI 自己维护一份"记忆文件"。但如果这份文件孤立存在，它就是一座孤岛——AI 记得项目名叫什么，但不知道需求文档在哪。需求文档记得功能列表，但不知道是谁提的、为什么提。

**Obsidian 解决的就是这个问题。**

```
你的 Obsidian Vault (hypone-node/)
│
├── claude-memory/                    ← AI 的工作记忆
│   ├── _index.md                     ← 每次对话加载，恢复 80% 上下文
│   ├── decisions.md                  ← 技术决策可追溯到 [[project]] 和 [[ADR-001]]
│   └── progress.md                   ← 当前进度，指向 [[knowledge-base/wiki/SRS初稿]]
│       │
│       └──────── [[双向链接]] ────────┐
│                                      │
├── knowledge-base/                   ← 项目的知识资产
│   ├── raw/notes/                    ← 涉众原始需求
│   │   └── 前台店员-20260623-需求记录.md  ← AI 生成的，人类审查的
│   └── wiki/
│       ├── summaries/SRS初稿.md       ← AI 写入，Obsidian 渲染
│       └── baselines/BL-20260623-01/  ← 基线版本，永久只读
```

### 三个关键价值

**1. 记忆与知识双向链接**
`decisions.md` 里写 "用 DeepSeek 替代 OpenAI" 不是一句孤立的话——它链接到 `[[environment]]`（那里记录了 OpenAI API 超时的诊断过程），链接到 `[[project]]`（那里记录了涉众对响应速度的要求）。下次做类似决策时，AI 读一条 ADR 就能遍历整个决策链。

**2. 人类可审查、可编辑**
所有记忆文件都是纯 Markdown。AI 写的决策记录，你可以在 Obsidian 里直接改、加注释、标废弃。这不是一个黑盒——它是你的 vault 的一部分。

**3. 零切换成本**
你在 Obsidian 里写需求文档、画架构图、做会议记录。AI 的上下文记忆放在同一个 vault 的 `claude-memory/` 子目录下。你不需要打开另一个工具、登录另一个服务、学习另一种格式。`Ctrl+O` → `_index` → 看到 AI 的完整记忆。

### 和其他"AI 记忆方案"的区别

| 方案 | 格式 | 可链接 | 人类可编辑 | 隐私 |
|---|---|---|---|---|
| Claude Code 内置 memory | 单个 Markdown | ❌ | ✅ | ✅ |
| LangChain Memory | JSON/向量数据库 | ❌ | ❌ | 看部署 |
| Continue.dev | JSON | ❌ | ❌ | ✅ |
| **claude-memory + Obsidian** | Markdown + `[[]]` | ✅ 双向 | ✅ | ✅ 全本地 |

---

## 触发时机

**读记忆（Tier 1 始终加载）：**
- 新会话开始，检测到对话涉及旧项目
- 用户说 "继续 XX 项目"、"上次那个"、"回到..."、"resume..."

**写记忆（增量更新）：**
- 首次接触新项目 → 脚手架 `_index.md` + `project.md`
- 任何非平凡的技术选择 → 追加 `decisions.md`（ADR 格式）
- 命令失败并找到原因 → 更新 `environment.md`
- 用户纠正你（路径/工具/习惯）→ 更新对应文件
- 完成一个里程碑 → 更新 `progress.md`
- 用户明确说 "记住这个"、"save this"、"更新记忆"

---

## 文件架构：6 文件 × 3 层级

```
claude-memory/               ← 放在 Obsidian vault 根目录下
│
├── TIER 1 — 始终加载 ─────────────────────
├── _index.md                ← 50-80行。项目名、当前阶段、关键决策一行、
│                               智能体/组件清单、环境亮点、恢复口令。
│                               用 [[wikilinks]] 链接到 Tier 2 文件。
│
├── TIER 2 — 按需加载 ─────────────────────
├── project.md               ← 项目元信息：涉众、仓库路径、目录结构
├── decisions.md             ← ADR 格式：编号、日期、决策、理由、替代方案
├── progress.md              ← 更新最频繁：checkbox 待办、产出物清单、上次运行状态
├── environment.md           ← 最有价值的文件：OS/Shell/PATH 问题、版本、API 配置
│
├── TIER 3 — 首次交互加载 ─────────────────
└── preferences.md           ← 交互风格、编码习惯、技术偏好
```

### 为什么分层而不是单文件？

单文件方案（`CLAUDE.md`）的致命缺陷：随着项目推进，文件从 50 行膨胀到 500 行。500 行塞进每次对话的 system prompt，其中 400 行是过时的进度记录和不再相关的决策细节。真正有用的 50 行被稀释了。

分层方案：`_index.md` 始终只有 50-80 行（摘要+链接），细节按需加载。下次对话先读 50 行恢复概览，需要了再 `Read decisions.md` 看具体 ADR。

---

## 各文件规范

### `_index.md` — 恢复 80% 上下文，60 秒内

**包含：**项目名、当前阶段、关键决策（一行决策+一行理由）、Agent/组件清单（名称+职责+文件）、环境亮点、恢复口令

**不包含：**超过一行的解释、完整 ADR、待办逐条（那是 progress.md 的事）

**模板：** → `references/template-index.md`

### `decisions.md` — 每条决策一个 ADR

```markdown
## ADR-001: LLM 选择 DeepSeek
- **日期**: 2026-06-23 | **状态**: 已采纳
- **决策**: 使用 DeepSeek deepseek-chat 作为所有 Agent 的 LLM
- **理由**: 国内直连；crewAI 原生支持；成本低
- **替代**: GPT-4o（网络不通）；代理中转（未配置）
- **来源**: 实测 OpenAI API timeout，DeepSeek 正常
```

**关键原则：**不仅记录"选了 A"，还记录"为什么没选 B 和 C"。三个月后回来看，你才能判断当时的决策前提是否还成立。

### `environment.md` — 踩坑记录，最有 ROI 的文件

不仅要写"python 用不了"，还要写**诊断过程和解决方案**：

```markdown
## Windows Git Bash — python 命令问题
- **现象**: `python` 命令 exit code 49，无任何输出
- **原因**: Git Bash PATH 中 Python312 目录未被正确引用
- **解决**: 用 `py` 替代 `python`；或 `py -3`
- **影响范围**: 所有 Bash 工具调用的 Python 命令
```

### `progress.md` — 更新最频繁，最易过时

结束时必须更新三件事：已完成（checkbox）、待完成（checkbox）、产出物清单（文件名+行数+状态）

---

## 工作流

```
会话开始
  ├─ Read claude-memory/_index.md          （始终）
  ├─ Read claude-memory/progress.md         （如果是继续项目）
  ├─ Read claude-memory/environment.md      （如果要跑命令）
  └─ Read claude-memory/decisions.md        （如果要改架构）

会话中
  ├─ 每个重大决策 → 追加 decisions.md
  ├─ 每个里程碑   → 更新 progress.md
  └─ 每次被纠正   → 更新对应文件

会话结束
  ├─ 刷新 progress.md（待办 + 产出物清单）
  ├─ 检查 _index.md 摘要是否过时
  └─ 告知用户本次更新了哪些文件
```

---

## 核心原则

1. **Layered, not monolithic.** 不要把一切都塞进一个文件。
2. **_index.md is the contract.** 没被它链接的内容，下个会话等于不存在。
3. **ADR for decisions, not notes.** 决策记录必须包含"为什么"和"其他选项为什么不选"。
4. **Progress rots fastest.** 每次会话结束必须更新。过时的进度比没有进度更危险。
5. **Environment quirks are gold.** "python 不行用 py" 这条信息，每次会话值 5 分钟。
6. **Obsidian links create a web, not a list.** 用 `[[wikilinks]]` 把 AI 记忆和项目知识织成一张网。当 AI 读一条 ADR 时，能顺着链接走到环境约束、项目背景、涉众需求。
7. **Don't record what the repo already records.** 代码结构、git 历史、config 文件已经是真相。记忆只记录代码推导不出的东西。
8. **Human-readable, human-editable.** AI 写的任何记忆，用户都该能在 Obsidian 里打开、看懂、修改。不引入 JSON、数据库、向量嵌入。

---

## 参考模板

- `references/template-index.md` — `_index.md` 骨架模板
- `references/template-adr.md` — 架构决策记录模板

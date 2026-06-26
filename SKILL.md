---
name: claude-memory
description: "Layered project memory system for persistent AI context across sessions. Designed to live inside an Obsidian vault — memory files and project knowledge interlink via [[wikilinks]]. Use when starting a project, recording decisions, tracking progress, documenting environment quirks, resuming after a break, or querying the knowledge base (使用知识库 / 查询知识库 / 查笔记 / 查需求 / knowledge base). Generates 6 structured markdown files with a lightweight _index.md (50-80 lines) for fast context restoration — no more re-explaining project background every session."
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
- 用户说 "使用知识库"、"查询知识库"、"查笔记"、"查需求"、"知识库" → 先检查 Claude Code 内置 memory 是否已有 vault 路径；若有则直接用，若无则读取 _index.md 定位，然后存入 Claude Code memory

**写记忆（增量更新）：**
- **用户首次告知 vault/知识库位置 → 必须写入 Claude Code 内置 memory**（`~/.claude/projects/{hash}/memory/knowledge-base-location.md`），含 vault 路径和目录结构，确保后续会话自动加载无需搜索
- 首次接触新项目 → 脚手架 `_index.md` + `project.md`
- 任何非平凡的技术选择 → 追加 `decisions.md`（ADR 格式）
- 命令失败并找到原因 → 更新 `environment.md`
- 用户纠正你（路径/工具/习惯）→ 更新对应文件
- 完成一个里程碑 → 更新 `progress.md`
- 用户明确说 "记住这个"、"save this"、"更新记忆"

---

## 快速开始：复制模板包

Skill 自带一套空模板在 `memory-package/` 目录。开新项目时：

```
1. 复制 memory-package/ → {obsidian_vault}/claude-memory/
2. 逐文件填入 {PLACEHOLDER} 占位符
3. _index.md 填完后，去掉不相关的骨架行
```

模板包包含 6 个带占位符的骨架文件，每个文件头都有注释说明填什么。

---

## 文件架构：多项目 × 3 层级

```
{obsidian_vault}/
└── claude-memory/                   ← 从 memory-package/ 复制而来
    │
    ├── _index.md                    ← TIER 1 全局：始终加载
    │   列出所有项目 + 全局环境摘要 + 当前焦点
    │
    ├── environment.md               ← TIER 2 全局：按需加载（写一次，所有项目共享）
    │   OS、Shell、命令坑、依赖版本、API 配置
    │
    ├── preferences.md               ← TIER 3 全局：首次交互加载（写一次）
    │   交互风格、编码习惯、技术偏好
    │
    ├── {project-slug-1}/            ← 项目 A
    │   ├── _index.md                ← TIER 1 项目级：按需加载
    │   ├── project.md               ← 涉众、目录结构、业务流程
    │   ├── decisions.md             ← ADR 格式决策记录
    │   └── progress.md              ← 待办 + 产出物 + 上次运行
    │
    └── {project-slug-2}/            ← 项目 B
        ├── _index.md
        ├── project.md
        ├── decisions.md
        └── progress.md
```

### 为什么共享 + 分项目？

| 文件 | 层级 | 理由 |
|---|---|---|
| `environment.md` | 全局共享 | 同一台机器的 OS/Shell/PATH 不会随项目变 |
| `preferences.md` | 全局共享 | 同一个人的交互风格不会随项目变 |
| `_index.md`（顶层） | 全局 | 快速定位「现在在搞哪个项目」 |
| `project.md` | 项目级 | 每个项目的涉众、目录、流程完全不同 |
| `decisions.md` | 项目级 | 每个项目的技术选型独立 |
| `progress.md` | 项目级 | 每个项目的进度独立 |
| `_index.md`（项目级） | 项目级 | 项目内快速恢复上下文 |

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
  ├─ 检查 Claude Code 内置 memory 是否已有 vault 路径  ← 新增：一步命中
  ├─ （若无）Read claude-memory/_index.md              （始终 — 全局索引）
  ├─ （若无）Read claude-memory/environment.md          （如果要跑命令）
  ├─ Read claude-memory/{project}/_index.md     （项目级索引）
  ├─ Read claude-memory/{project}/progress.md   （如果是继续项目）
  └─ Read claude-memory/{project}/decisions.md  （如果要改架构）

会话中
  ├─ 首次确认 vault 路径 → 写入 Claude Code 内置 memory  ← 新增
  ├─ 每个重大决策 → 追加 {project}/decisions.md
  ├─ 每个里程碑   → 更新 {project}/progress.md
  ├─ 每次被纠正   → 更新对应文件
  └─ 新项目启动   → 复制 memory-package/{project-slug}/ → 新子目录

会话结束
  ├─ 刷新 {project}/progress.md
  ├─ 刷新 claude-memory/_index.md 项目表
  └─ 告知用户本次更新了哪些文件
```

### 两层记忆协作

| 记忆层 | 位置 | 职责 |
|--------|------|------|
| Claude Code 内置 memory | `~/.claude/projects/{hash}/memory/` | 快速定位：vault 路径、知识库位置、用户关键偏好。**跨会话自动加载，无需读文件** |
| Obsidian vault 记忆 | `{vault}/claude-memory/` | 深度记忆：项目进度、ADR、环境踩坑、完整偏好。**按需读取，双向链接** |

**关键规则：vault 路径必须同时存在于两层。** 内置 memory 保证下次会话秒找到 vault；vault 内 `_index.md` 保证 Obsidian 用户也能看到完整上下文。

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

---
name: pair-grow
description: "Human-AI pair growth system. Tracks what both sides learn from every interaction — bugs, communication patterns, skill evolution, AI blindspots, and reusable success templates. AI-agnostic: works with Claude, ChatGPT, Copilot, or any AI. Designed to live alongside claude-memory in the same Obsidian vault, with [[wikilinks]] connecting growth records to project context. Use when a bug teaches something, when communication misfires, when the user demonstrates new technical understanding, when AI repeats a mistake, or when a complex task finally succeeds through iteration."
---

# Pair Grow — Human-AI Growth System

## 哲学

AI 编程不是工具使用，是**结对关系**。

结对编程中，两个人都应该从合作中成长。但当前的 AI 编程助手没有长期记忆——每一次新对话，你和 AI 都是从零开始。你学会了更清晰地描述需求，AI 不知道。AI 发现了你的表达习惯，下个会话忘了。Bug 出现了，修好了，但根因分析丢失了。三个月后同类 bug 再次出现，AI 从头解释你早已理解的概念。

**pair-grow 解决这个问题。** 它记录了"谁学到了什么"——不只是 AI 的输出质量在提升，而是你的需求和表达能力在进化，以及 AI 对你的理解在加深。

### 为什么 AI-agnostic（适配任何 AI）

你可能今天用 Claude Code，明天用 ChatGPT，后天用 Copilot。但**你是同一个你**——你的技术知识、需求表达习惯、常见误解模式，不应该因为换了一个 AI 助手就全部重来。

pair-grow 的设计原则：**记录属于人，不属于具体 AI。** 

- `skill-map.md` 记录的是**你的**技术知识增长，不是 Claude 的
- `comm-patterns.md` 记录的是**你的**表达习惯，任何 AI 读到都能适配
- `bug-journal.md` 记录的是**你和 AI 协作中**出现的问题，以及谁从中学到了什么

你在 Obsidian vault 里装了 pair-grow。今天用 Claude Code，它读 `pair-grow/_index.md` 恢复对你的了解。明天用另一个 AI，你把 vault 里的 pair-grow 文件给它——它就能立即知道你的技术水平、沟通习惯、以及历史上踩过的坑。

---

## 和 claude-memory 的关系

```
{obsidian_vault}/
│
├── claude-memory/           ← "我们在做什么"（项目维度）
│   ├── _index.md                项目列表、全局环境
│   └── {project}/               _index, project, decisions, progress
│       │
│       └─── [[wikilinks]] ──── 双向链接，各司其职
│                                    │
├── pair-grow/               ← "我们怎么越做越好"（关系维度）
│   ├── _index.md                成长概览、当前能力评估
│   ├── bug-journal.md           有学习价值的 bug
│   ├── skill-map.md             用户技术知识树
│   ├── comm-patterns.md         沟通模式词典
│   ├── ai-blindspots.md         AI 已知易错领域
│   └── success-templates.md     可复制的成功模式
│
└── knowledge-base/          ← 项目知识资产
```

**分工明确：** `claude-memory` 告诉你"打印店项目做到需求分析阶段了"；`pair-grow` 告诉你"用户已经不需要解释参数化查询了，上次沟通中他说'快'通常指 2 秒以内，AI 在 Windows 路径处理上容易出错需要自检"。

**互相引用：** bug-journal 里一条 "SQL 注入 bug 修复" 可以链接到 claude-memory 里的 `[[print-shop/decisions]]`（该 bug 涉及 ADR 技术选型）。反过来，progress 里的一个复杂任务链接到 `[[../pair-grow/success-templates]]`（这个模板加速了下一次同类任务）。

---

## 触发时机

| 文件 | 写入时机 | 读取时机 |
|---|---|---|
| `bug-journal.md` | bug 修复后，**且**有学习价值（新类型、新根因、新解法） | 遇到类似错误信息时 |
| `skill-map.md` | 用户表现出理解了一个新技术概念，或反复问同类问题时 | 每次对话开始——调整解释深度 |
| `comm-patterns.md` | 发生误解并被澄清后 | 收到模糊需求时——查词典消歧 |
| `ai-blindspots.md` | AI 同类错误出现第 2 次（第 1 次是意外，第 2 次是 pattern） | 在相关领域生成代码后——自检 |
| `success-templates.md` | 复杂任务经过 ≥3 轮迭代最终成功 | 遇到类似任务类型时——优先匹配模板 |

---

## 文件设计

### 1. `_index.md` — 成长概览

始终加载（40-60行）。一眼看到当前协作状态。

**必须包含：**
- 用户技能概览（已掌握 / 学习中 / 待接触）
- AI 当前已知盲区数量
- 最近有学习价值的 bug（最近 3-5 条）
- 沟通模式摘要（2-3 条最重要的）
- 成功模板清单

```markdown
# Pair Grow Index

## 用户技能概览
- 已掌握: SQL参数化查询, Python装饰器, RESTful设计
- 学习中: 异步编程, Docker编排
- 待接触: Kubernetes, 消息队列

## AI 盲区（5个）
- Windows路径处理, GBK编码, ...

## 最近 Bug
| 日期 | Bug类型 | 根因方向 | 学到什么 |
|---|---|---|---|
| 0623 | SQL注入 | AI→用户 | 用户已理解参数化查询 |
```

### 2. `bug-journal.md` — 有学习价值的 Bug

**不是所有 bug 都记。** 记的条件：
- 暴露了某一方的知识盲区
- 根因不是简单的"写错了"而是"理解偏差"
- 修复过程中双方学到了新东西
- 同类型 bug 首次出现（第二次出现说明第一次没学会）

```markdown
## BUG-003: SQL 注入 — 用户未说明输入来源
- **日期**: 2026-06-23
- **症状**: `process_user_data` 字符串拼接SQL
- **根因方向**: AI→用户 + 用户→AI（双向）
  - AI 没追问 `user_id` 来自哪里
  - 用户没意识到需要说明"这是用户输入"
- **修复**: 参数化查询
- **用户学到**: 描述需求时标注"此值来自用户输入"
- **AI 学到**: 看到 `= " + var` 模式必须追问输入来源
- **关联**: [[print-shop/decisions#ADR-006]]
- **标签**: #sql #security #communication
```

### 3. `skill-map.md` — 用户技术知识树

记录用户的真实技术水平，**实时更新、渐进精细**。

初始是空的。随着对话积累，AI 观察用户的行为来填充：
- 用户说"不用解释这个" → 标记为已掌握
- 用户问"为什么要用这个" → 标记为学习中
- 用户表现出困惑 → 标记为待学习
- 某主题连续 3 次不出现在困惑中 → 从学习中提升到已掌握

```markdown
## Python
### 已掌握
- 基础语法, 函数, 类, 装饰器, 列表推导
- pip, 虚拟环境

### 学习中
- 异步 (asyncio) — 2026-06-20 开始接触
- 类型注解 — 2026-06-22 开始使用

### 待接触
- C扩展, GIL深入

## 数据库
### 已掌握
- SQL基础, 参数化查询 (2026-06-23 掌握)
- SQLite

### 学习中
- PostgreSQL索引优化
```

**关键原则：skill-map 不由用户自己填，由 AI 观察推断。** 用户往往不知道自己不知道什么，或者不知道自己其实已经懂了。AI 通过对话中的信号来判断——解释一个概念时用户说"懂了"，还是追问"为什么"，还是沉默（需要换个方式解释）。

### 4. `comm-patterns.md` — 沟通模式词典

记录用户特有的表达方式和真实意图之间的映射。

```markdown
## 已确认的映射
| 用户说 | 通常指 | 需要确认 |
|---|---|---|
| "把这个改快点" | 响应时间 < 2秒 | 数据量级？ |
| "做个简单的登录" | 用户名+密码，不需要OAuth/2FA | 是否需要记住我？ |
| "和之前一样" | 指上一个完成的功能的模式 | 确认具体是哪个功能 |

## 已避免的误解
- "部署" — 不是Docker/K8s，是FTP上传到虚拟主机
- "测试" — 不是单元测试，是手动跑一遍看结果
```

### 5. `ai-blindspots.md` — AI 的已知盲区

每个 AI（包括 Claude）都有系统性弱项——某些领域错误率显著高于平均水平。这个文件追踪当前使用的 AI 的"已知弱点"。

```markdown
## Windows 路径处理
- **症状**: 正斜杠 vs 反斜杠混淆；盘符未处理
- **触发条件**: 涉及文件路径 + Windows 环境
- **自检步骤**: 
  1. 所有路径使用 `pathlib.Path` 
  2. 检查 `C:` 盘符是否正确
  3. 检查 GBK 编码
- **出现次数**: 3
- **关联**: [[环境/environment#Windows-Git-Bash]]

## GBK 编码
- **症状**: emoji/特殊字符写入文件时报 UnicodeEncodeError
- **触发条件**: Windows + 中文输出
- **自检步骤**: 在文件头添加 `sys.stdout = io.TextIOWrapper(...)`
- **出现次数**: 2
```

### 6. `success-templates.md` — 可复制的成功模式

当一个复杂任务经过多次迭代最终成功时，提取"这次为什么最终对了"。

```markdown
## TEMPLATE-001: 多Agent系统完整设计
- **任务类型**: 从需求文档生成全量Agent配置
- **适用场景**: 用户提供结构化需求文档，需要设计多个Agent
- **成功迭代**: 8轮
- **关键转折点**: 
  - 第3轮：从"直接写代码"转为"先讨论架构再写" → 减少返工
  - 第5轮：发现用户需求文档有未读的第8节 → 补充完整设计
- **有效 prompt 结构**:
  1. 先逐段分析需求文档（不写代码）
  2. 给出Agent矩阵概述（表格）
  3. 用户确认 → 写agents.yaml
  4. 用户确认 → 写tasks.yaml → flow.py
- **适用模型的建议**: 复杂推理用DeepSeek/deepseek-chat足够
```

---

## 工作流

```
新会话开始
  ├─ Read pair-grow/_index.md              （始终 — 成长概览）
  ├─ Read pair-grow/skill-map.md            （调整解释深度）
  ├─ Read pair-grow/comm-patterns.md        （消歧用户表达）
  └─ Read pair-grow/ai-blindspots.md        （自检当前任务是否涉险）

Bug 出现并修复后
  └─ 判断是否有学习价值 → 是 → 追加 bug-journal.md
                             → 更新 skill-map.md（如果用户学到了）
                             → 更新 ai-blindspots.md（如果是新 pattern）

误解被澄清后
  └─ 追加 comm-patterns.md

复杂任务成功后（≥3轮迭代）
  └─ 提取 → 追加 success-templates.md

用户表现出理解/困惑
  └─ 更新 skill-map.md
```

---

## 和 claude-memory 的协作示例

```
场景: 用户要加一个新功能"订单超时自动取消"

1. AI 读 claude-memory/print-shop/_index.md
   → 知道在需求工程阶段，还没到编码
   
2. AI 读 pair-grow/_index.md
   → 用户已掌握"异步任务"概念
   → comm-patterns 中"自动"通常指"后台定时任务，不是用户手动点"
   
3. AI 读 pair-grow/ai-blindspots.md
   → Windows 定时任务配置是盲区 → 用 APScheduler 而非 Windows Task Scheduler
   
4. AI 给出建议，跳过对"什么是异步"的解释
   → 用户说"懂了" → skill-map 确认"异步编程" → 已掌握
   
5. 如果过程中出现 bug：
   → bug-journal 追加
   → 链接到 claude-memory/print-shop/decisions#ADR-009（n8n调度相关）
```

---

## 通用性设计

本 skill 刻意避免绑定特定 AI 工具。所有文件是纯 Markdown。一个用户可能：
- 今天用 Claude Code 写代码
- 明天用 ChatGPT 讨论架构
- 后天用 Copilot 做重构

每次切换 AI 时，用户只需把 vault 中的 `pair-grow/` 文件提供给新 AI（或保持同一 vault）。

**不同 AI 读取 pair-grow 时都能受益于：**
- `skill-map.md` → 调整解释深度
- `comm-patterns.md` → 理解用户的特殊表达
- `bug-journal.md` → 避免重复踩坑
- `ai-blindspots.md` → 不同 AI 有不同的盲区，可以各自维护

**注意：** `ai-blindspots.md` 可能因 AI 而异。建议结构：同一文件，按 AI 型号分段。Claude 提 Windows 路径问题，ChatGPT 可能提其他盲区。切换 AI 时盲区表会被新 AI 读取并扩展。

---

## 参考模板

- `memory-package/` — 空模板包（复制到新 vault 开箱即用）
- `references/` — 各文件的详细写作指南

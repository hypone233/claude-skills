# _index.md Template

```markdown
# Claude Memory Index

> 轻量索引 — 每次对话自动加载。深入细节时按需读取具体文件。

## 当前项目

- **项目名**: {project_name}
- **代码路径**: `{repo_path}`
- **知识库路径**: `{kb_path}`
- **当前阶段**: {current_phase}
- **进度详情**: → [[progress]]

## 关键决策

| 决策 | 选择 | 理由 |
|---|---|---|
| {decision_1_title} | {decision_1_choice} | {decision_1_rationale_one_line} |
| {decision_2_title} | {decision_2_choice} | {decision_2_rationale_one_line} |
| 详情 | → [[decisions]] | |

## 环境约束

- OS: {os}
- Shell: {shell}（{command_quirk}）
- {key_dependency}: {version}
- {api_provider_note}
- 详情: → [[environment]]

## 用户偏好

- {preference_1}
- {preference_2}
- 详情: → [[preferences]]

## {component_list_title}

| {col1_header} | {col2_header} | {col3_header} | {col4_header} |
|---|---|---|---|
| {rows...} |

## 快速恢复

下次对话只需说：「{resume_phrase}」
```

# Skills Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2026%2C%202026%201%3A09%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.119-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-skills-implementation.md)

Claude Code skills — frontmatter 字段和官方内置技能。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</td>
</tr>
</table>

---

## Frontmatter Fields (15)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | 显示名称和 `/slash-command` 标识符。如果省略，默认为目录名 |
| `description` | string | Recommended | 技能的作用。显示在自动补全中，供 Claude 用于自动发现 |
| `when_to_use` | string | No | 关于 Claude 何时应调用该技能的额外上下文 — 触发短语和请求示例。附加到 `description` 之后显示在技能列表中，计入 1,536 字符上限 |
| `argument-hint` | string | No | 自动补全期间显示的提示（如 `[issue-number]`、`[filename]`） |
| `arguments` | string/list | No | 用于在技能内容中进行 `$name` 替换的命名位置参数。接受以空格分隔的字符串或 YAML 列表 — 名称按顺序映射到参数位置 |
| `disable-model-invocation` | boolean | No | 设为 `true` 可阻止 Claude 自动调用此技能 |
| `user-invocable` | boolean | No | 设为 `false` 可从 `/` 菜单中隐藏 — 技能仅作为背景知识使用，适用于 agent 预加载 |
| `allowed-tools` | string | No | 此技能处于活动状态时无需权限提示即可使用的工具 |
| `model` | string | No | 运行此技能时使用的模型（如 `haiku`、`sonnet`、`opus`） |
| `effort` | string | No | 调用时覆盖模型的 effort 级别（`low`、`medium`、`high`、`xhigh`、`max`） |
| `context` | string | No | 设为 `fork` 可在隔离的 subagent 上下文中运行技能 |
| `agent` | string | No | 当 `context: fork` 时使用的 subagent 类型（默认为 `general-purpose`） |
| `hooks` | object | No | 限定在此技能范围内的生命周期 hooks |
| `paths` | string/list | No | 限制技能自动激活范围的 glob 模式。接受逗号分隔的字符串或 YAML 列表 — Claude 仅在处理匹配文件时加载该技能 |
| `shell` | string | No | `` !`command` `` 块使用的 shell — `bash`（默认）或 `powershell`。需要设置 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](../!/tags/official.svg) **内置技能 (5)**

| # | Skill | Description |
|---|-------|-------------|
| 1 | `simplify` | 检查变更的代码以复用、质量和效率 — 通过重构消除重复代码 |
| 2 | `batch` | 在多个文件上批量运行命令 |
| 3 | `debug` | 调试失败的命令或代码问题 |
| 4 | `loop` | 按固定间隔循环运行提示或 slash command（最长 3 天） |
| 5 | `claude-api` | 使用 Claude API 或 Anthropic SDK 构建应用 — 在 `anthropic` / `@anthropic-ai/sdk` 导入时触发 |

另见：[Official Skills Repository](https://github.com/anthropics/skills/tree/main/skills) 社区维护的可安装技能仓库。

---

## 来源

- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

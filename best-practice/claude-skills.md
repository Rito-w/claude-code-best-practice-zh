<!-- TRANSLATED: Auto-translated from source -->

# Skills Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2028%2C%202026%205%3A59%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-skills-implementation.md)

Claude Code skills — frontmatter fields 和 official bundled skills。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (13)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Display name 和 `/slash-command` identifier。如果省略则默认为 directory name |
| `description` | string | Recommended | Skill 的作用。在 autocomplete 中显示并被 Claude 用于 auto-discovery |
| `argument-hint` | string | No | 在 autocomplete 期间显示的 hint（例如 `[issue-number]`, `[filename]`） |
| `disable-model-invocation` | boolean | No | 设置为 `true` 以防止 Claude 自动调用此 skill |
| `user-invocable` | boolean | No | 设置为 `false` 以在 `/` menu 中隐藏 — skill 变为仅 background knowledge，用于 agent preloading |
| `allowed-tools` | string | No | 当此 skill 激活时允许使用的 Tools（无需 permission prompts） |
| `model` | string | No | 当此 skill 运行时使用的 Model（例如 `haiku`, `sonnet`, `opus`） |
| `effort` | string | No | 调用时覆盖 model effort level（`low`, `medium`, `high`, `max`） |
| `context` | string | No | 设置为 `fork` 以在 isolated subagent context 中运行 skill |
| `agent` | string | No | 当设置 `context: fork` 时的 Subagent type（默认：`general-purpose`） |
| `hooks` | object | No | 作用域为此 skill 的 Lifecycle hooks |
| `paths` | string/list | No | Glob patterns 限制 skill 何时自动激活。接受逗号分隔的 string 或 YAML list — Claude 仅在处理匹配文件时加载 skill |
| `shell` | string | No | `` !`command` `` blocks 的 Shell — `bash`（默认）或 `powershell`。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](!/tags/official.svg) **(5)**

| # | Skill | Description |
|---|-------|-------------|
| 1 | `simplify` | 审查变更代码的 reuse, quality, 和 efficiency — 重构以消除 duplication |
| 2 | `batch` | 批量跨多个文件运行 commands |
| 3 | `debug` | Debug failing commands 或 code issues |
| 4 | `loop` | 在 recurring interval 上运行 prompt 或 slash command（最多 3 天） |
| 5 | `claude-api` | 使用 Claude API 或 Anthropic SDK 构建 apps — 在 `anthropic` / `@anthropic-ai/sdk` imports 时触发 |

另见：[Official Skills Repository](https://github.com/anthropics/skills/tree/main/skills) 获取 community-maintained installable skills。

---

## Sources

- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

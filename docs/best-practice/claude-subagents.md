<!-- TRANSLATED: Auto-translated from source -->

# Sub-agents Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2028%2C%202026%206%3A00%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-subagents-implementation.md)

Claude Code subagents — frontmatter fields 和 official built-in agent types。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (16)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | 使用小写字母和连字符的唯一 identifier |
| `description` | string | Yes | When to invoke。使用 `"PROACTIVELY"` 表示 Claude 的 auto-invocation |
| `tools` | string/list | No | Tools 的逗号分隔 allowlist（例如 `Read, Write, Edit, Bash`）。如果省略则继承所有 tools。支持 `Agent(agent_type)` syntax 以限制可 spawn 的 subagents；旧版 `Task(agent_type)` alias 仍然有效 |
| `disallowedTools` | string/list | No | 要拒绝的 Tools，从 inherited 或 specified list 中移除 |
| `model` | string | No | Model alias：`haiku`, `sonnet`, `opus`, 或 `inherit`（默认：`inherit`） |
| `permissionMode` | string | No | Permission mode：`default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, 或 `plan` |
| `maxTurns` | integer | No | Subagent 停止前的 Maximum number of agentic turns |
| `skills` | list | No | 启动时预加载到 agent context 的 Skill names（完整内容 injected，不只是 made available） |
| `mcpServers` | list | No | 此 subagent 的 MCP servers — server name strings 或 inline `{name: config}` objects |
| `hooks` | object | No | 作用域为此 subagent 的 Lifecycle hooks。所有 hook events 都支持；`PreToolUse`, `PostToolUse`, 和 `Stop` 是最常见的 |
| `memory` | string | No | Persistent memory scope：`user`, `project`, 或 `local` |
| `background` | boolean | No | 设置为 `true` 以始终作为 background task 运行（默认：`false`） |
| `effort` | string | No | 当此 subagent 激活时的 Effort level override：`low`, `medium`, `high`, `max`。默认：从 session 继承 |
| `isolation` | string | No | 设置为 `"worktree"` 以在 temporary git worktree 中运行（如果没有 changes 则自动清理） |
| `initialPrompt` | string | No | 当此 agent 作为 main session agent 运行时（通过 `--agent` 或 `agent` setting）自动作为第一个 user turn 提交。Commands 和 skills 被处理。Prepended 到任何 user-provided prompt |
| `color` | string | No | 用于 visual distinction 的 CLI output color（例如 `green`, `magenta`）。Functional 但不在 official frontmatter table 中 — 仅在 interactive quickstart 中有文档 |

---

## ![Official](_media/tags/official.svg) **(6)**

| # | Agent | Model | Tools | Description |
|---|-------|-------|-------|-------------|
| 1 | `general-purpose` | inherit | All | Complex multi-step tasks — research, code search, 和 autonomous work 的默认 agent type |
| 2 | `Explore` | haiku | Read-only（无 Write, Edit） | Fast codebase search and exploration — 优化用于 finding files, searching code, 和 answering codebase questions |
| 3 | `Plan` | inherit | Read-only（无 Write, Edit） | Plan mode 中的 Pre-planning research — 在编写代码之前 explores the codebase 并 designs implementation approaches |
| 4 | `Bash` | inherit | Bash | 在 separate context 中 Running terminal commands |
| 5 | `statusline-setup` | sonnet | Read, Edit | 配置用户的 Claude Code status line setting |
| 6 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | 回答关于 Claude Code features, Agent SDK, 和 Claude API 的问题 |

---

## Sources

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [CLI reference — Claude Code Docs](https://code.claude.com/docs/en/cli-reference)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

# Sub-agents 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2026%2C%202026%201%3A10%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.119-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-subagents-implementation.md)

Claude Code subagents — frontmatter 字段和官方内置 agent 类型。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter 字段 (16)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | 是 | 唯一标识符，使用小写字母和连字符 |
| `description` | string | 是 | 何时调用。使用 `"PROACTIVELY"` 表示由 Claude 自动调用 |
| `tools` | string/list | 否 | 工具白名单，逗号分隔（如 `Read, Write, Edit, Bash`）。省略时继承所有工具。支持 `Agent(agent_type)` 语法限制可衍生的 subagent；旧版 `Task(agent_type)` 别名仍可用 |
| `disallowedTools` | string/list | 否 | 要拒绝的工具，从继承或指定列表中移除 |
| `model` | string | 否 | 使用的模型: `sonnet`, `opus`, `haiku`，完整模型 ID（如 `claude-opus-4-6`），或 `inherit`（默认: `inherit`） |
| `permissionMode` | string | 否 | 权限模式: `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, 或 `plan` |
| `maxTurns` | integer | 否 | subagent 停止前的最大 agentic 轮数 |
| `skills` | list | 否 | 启动时预加载到 agent 上下文的 Skill 名称（注入完整内容，而非仅使其可用） |
| `mcpServers` | list | 否 | 此 subagent 的 MCP 服务器 — 服务器名称字符串或内联 `{name: config}` 对象 |
| `hooks` | object | 否 | 作用于此 subagent 的生命周期 Hook。所有 Hook事件均支持；`PreToolUse`, `PostToolUse`, `Stop` 最常用 |
| `memory` | string | 否 | 持久化记忆范围: `user`, `project`, 或 `local` |
| `background` | boolean | 否 | 设为 `true` 始终作为后台任务运行（默认: `false`） |
| `effort` | string | 否 | 此 subagent 激活时的 effort 级别覆盖: `low`, `medium`, `high`, `max`（仅 Opus 4.6）。默认: 继承自会话 |
| `isolation` | string | 否 | 设为 `"worktree"` 在临时 git worktree 中运行（无变更时自动清理） |
| `initialPrompt` | string | 否 | 此 agent 作为主会话 agent 运行时（通过 `--agent` 或 `agent` 设置）自动作为首个用户 prompt 提交。Command 和 skill 会被处理。附加在用户提供的 prompt 之前 |
| `color` | string | 否 | 任务列表和 transcript 中 subagent 的显示颜色: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, 或 `cyan` |

---

## ![Official](../!/tags/official.svg) **官方内置 Agent (5)**

| # | Agent | Model | Tools | 说明 |
|---|-------|-------|-------|-----|
| 1 | `general-purpose` | inherit | 全部 | 复杂的多步骤任务 — 默认的 agent 类型，适用于研究、代码搜索和自主工作 |
| 2 | `Explore` | haiku | 只读（无 Write, Edit） | 快速代码库搜索和探索 — 优化用于查找文件、搜索代码和回答代码库问题 |
| 3 | `Plan` | inherit | 只读（无 Write, Edit） | plan mode 下的预规划研究 — 探索代码库并设计实现方案，在编写代码之前 |
| 4 | `statusline-setup` | sonnet | Read, Edit | 配置用户的 Claude Code 状态行设置 |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | 回答关于 Claude Code 功能、Agent SDK 和 Claude API 的问题 |

---

## 来源

- [创建自定义 subagents — Claude Code 文档](https://code.claude.com/docs/en/sub-agents)
- [CLI 参考 — Claude Code 文档](https://code.claude.com/docs/en/cli-reference)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

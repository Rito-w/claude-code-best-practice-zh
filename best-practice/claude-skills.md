# Skills Best Practice

<<<<<<< HEAD
![Last Updated](https://img.shields.io/badge/Last_Updated-Jun%2012%2C%202026%2010%3A07%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.175-blue?style=flat&labelColor=555)<br>
=======
![Last Updated](https://img.shields.io/badge/Last_Updated-Jun%2013%2C%202026%2010%3A07%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.176-blue?style=flat&labelColor=555)<br>
>>>>>>> upstream/main
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-skills-implementation.md)

Claude Code skills — frontmatter fields and official bundled skills.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (16)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Display name and `/slash-command` identifier. Defaults to the directory name if omitted |
| `description` | string | Recommended | What the skill does. Shown in autocomplete and used by Claude for auto-discovery |
| `when_to_use` | string | No | Additional context for when Claude should invoke the skill — trigger phrases and example requests. Appended to `description` in the skill listing, counts toward the 1,536-character cap |
| `argument-hint` | string | No | Hint shown during autocomplete (e.g., `[issue-number]`, `[filename]`) |
| `arguments` | string/list | No | Named positional arguments for `$name` substitution in the skill content. Accepts a space-separated string or a YAML list — names map to argument positions in order |
| `disable-model-invocation` | boolean | No | Set `true` to prevent Claude from automatically invoking this skill |
| `user-invocable` | boolean | No | Set `false` to hide from the `/` menu — skill becomes background knowledge only, intended for agent preloading |
| `allowed-tools` | string | No | Tools allowed without permission prompts when this skill is active |
<<<<<<< HEAD
| `disallowed-tools` | string/list | No | 技能激活时从 Claude 可用工具池中移除的工具（例如为背景循环阻止 `AskUserQuestion`）。接受空格/逗号分隔的字符串或 YAML 列表——限制在下一条消息时清除 |
=======
| `disallowed-tools` | string/list | No | Tools removed from Claude's available pool while the skill is active (e.g. block `AskUserQuestion` for a background loop). Accepts a space/comma-separated string or YAML list — the restriction clears on the next message |
>>>>>>> upstream/main
| `model` | string | No | Model to use when this skill runs (e.g., `haiku`, `sonnet`, `opus`) |
| `effort` | string | No | Override the model effort level when invoked (`low`, `medium`, `high`, `xhigh`, `max`) |
| `context` | string | No | Set to `fork` to run the skill in an isolated subagent context |
| `agent` | string | No | Subagent type when `context: fork` is set (default: `general-purpose`) |
| `hooks` | object | No | Lifecycle hooks scoped to this skill |
| `paths` | string/list | No | Glob patterns that limit when the skill auto-activates. Accepts a comma-separated string or YAML list — Claude loads the skill only when working with matching files |
| `shell` | string | No | Shell for `` !`command` `` blocks — `bash` (default) or `powershell`. Requires `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](../!/tags/official.svg) **(10)**

| # | Skill | Description |
|---|-------|-------------|
| 1 | `code-review` | Review the current diff for correctness bugs at a chosen effort level (low/medium: fewer, high-confidence findings; high→max: broader coverage) — `--comment` posts findings as inline PR comments |
| 2 | `batch` | Run commands across multiple files in bulk |
| 3 | `debug` | Debug failing commands or code issues |
| 4 | `loop` | Run a prompt or slash command on a recurring interval (up to 3 days) |
| 5 | `claude-api` | Build apps with the Claude API or Anthropic SDK — triggers on `anthropic` / `@anthropic-ai/sdk` imports |
| 6 | `fewer-permission-prompts` | Scan transcripts for common read-only Bash/MCP calls and add a prioritized allowlist to `.claude/settings.json` to reduce permission prompts |
<<<<<<< HEAD
| 7 | `run` | 启动并驱动项目应用，以便在实际应用中查看变更效果（而非仅依赖测试）。需要 v2.1.145 |
| 8 | `verify` | 构建并运行应用以确认代码变更达到预期效果，而不回退到测试或类型检查。需要 v2.1.145 |
| 9 | `run-skill-generator` | 教会 `/run` 和 `/verify` 如何构建和启动项目 — 在 `.claude/skills/run-<name>/` 中记录每个项目的启动方案。需要 v2.1.145 |
| 10 | `simplify` | 审查变更代码的清理机会（复用、简化、效率、抽象层级），四个审查代理并行运行。从 v2.1.154 起**不再**查找正确性 bug——请使用 `/code-review` |
=======
| 7 | `run` | Launch and drive the project's app to see a change working in the real app (not just tests). Requires v2.1.145 |
| 8 | `verify` | Build and run the app to confirm a code change does what it should, without falling back to tests or type checks. Requires v2.1.145 |
| 9 | `run-skill-generator` | Teaches `/run` and `/verify` how to build and launch the project — records a per-project launch recipe at `.claude/skills/run-<name>/`. Requires v2.1.145 |
| 10 | `simplify` | Review changed code for cleanup opportunities (reuse, simplification, efficiency, abstraction level), four review agents in parallel. From v2.1.154 it does **not** hunt for correctness bugs — use `/code-review` for that |
>>>>>>> upstream/main

See also: [Official Skills Repository](https://github.com/anthropics/skills/tree/main/skills) for community-maintained installable skills.

---

## Sources

- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)


# Subagents Report — Changelog History（Subagents 报告 — 变更日志历史）

## 状态图例

| 状态 | 含义 |
|--------|---------|
| ✅ `COMPLETE (reason)` | 操作已成功执行并解决 |
| ❌ `INVALID (reason)` | 发现不正确、不适用或有意为之 |
| ✋ `ON HOLD (reason)` | 操作已推迟 — 等待外部依赖或用户决策 |

---

## [2026-02-28 03:22 PM PKT] Claude Code v2.1.63

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Agents Table | 在 "Agents in This Repository" 表格中添加 `workflow-changelog-claude-agents-frontmatter-agent` | ✅ COMPLETE (added with model: opus, inherits all tools, no skills/memory) |
| 2 | HIGH | Agents Table | 修复 presentation-curator skills 列 — 为 skill 名称添加 `presentation/` 前缀 | ✅ COMPLETE (updated to presentation/vibe-to-agentic-framework etc.) |
| 3 | MED | Field Documentation | 在 `color` 字段添加说明：功能正常但不在官方 frontmatter 表格中 | ✅ COMPLETE (added note about unofficial status in description column) |
| 4 | MED | Invocation Section | 扩展 invocation 部分，包含 --agents CLI flag, /agents command, claude agents CLI, agent resumption | ✅ COMPLETE (added invocation methods table with 5 methods) |

---

## [2026-03-07 08:35 AM PKT] Claude Code v2.1.71

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Broken Link | 修复 agent memory 链接到 `reports/claude-agent-memory.md` | ✅ COMPLETE |
| 2 | HIGH | Changed Behavior | 更新 `tools` 字段描述：`Task(agent_type)` → `Agent(agent_type)`（v2.1.63 重命名） | ✅ COMPLETE |
| 3 | HIGH | Changed Behavior | 更新 invocation 部分：Task tool → Agent tool（v2.1.63 重命名） | ✅ COMPLETE (updated heading, code example, and added rename note) |
| 4 | HIGH | Example Update | 更新完整功能示例：`Task(monitor, rollback)` → `Agent(monitor, rollback)` | ✅ COMPLETE |
| 5 | HIGH | Built-in Agent | 在 Official Claude Agents 表格中添加 `Bash` agent（model: inherit, purpose: terminal commands in separate context） | ✅ COMPLETE (added to table) |
| 6 | HIGH | Agents Table | 在 "Agents in This Repository" 表格中添加 `workflow-concepts-agent`（model: opus, color: green） | ✅ COMPLETE |
| 7 | HIGH | Agents Table | 在 "Agents in This Repository" 表格中添加 `workflow-claude-settings-agent`（model: opus, color: yellow） | ✅ COMPLETE |
| 8 | MED | Built-in Agent | 修复 `statusline-setup` model: `inherit` → `Sonnet` | ✅ COMPLETE |
| 9 | MED | Built-in Agent | 修复 `claude-code-guide` model: `inherit` → `Haiku` | ❌ NOT APPLICABLE (removed from table) |
| 10 | MED | Agents Table | 修复 `weather-agent` color: `teal` → `green` | ✅ COMPLETE |
| 11 | MED | Invocation | 在 invocation methods 表格中添加 `--agent <name>` CLI flag | ✅ COMPLETE (added as first row in invocation methods table) |
| 12 | MED | Changed Behavior | 更新第 147 行文本："Task tool" → "Agent tool" 在 Official Claude Agents 表格标题中 | ✅ COMPLETE (user rewrote header text) |
| 13 | MED | Cross-File | 更新 CLAUDE.md：`Task(...)` → `Agent(...)` 引用（第 50-53 行，61 行） | ✅ COMPLETE (updated orchestration section and tools field description) |

---

## [2026-03-12 12:17 PM PKT] Claude Code v2.1.74

未检测到漂移 — 报告与官方文档完全同步。所有 13 个 frontmatter 字段和 6 个内置 agents 匹配。

---

## [2026-03-13 04:21 PM PKT] Claude Code v2.1.74

未检测到漂移 — 报告与官方文档完全同步。所有 13 个 frontmatter 字段和 6 个内置 agents 匹配。

---

## [2026-03-15 12:50 PM PKT] Claude Code v2.1.76

未检测到漂移 — 报告与官方文档完全同步。所有 13 个 frontmatter 字段和 6 个内置 agents 匹配。

---

## [2026-03-17 12:42 PM PKT] Claude Code v2.1.77

未检测到漂移 — 报告与官方文档完全同步。所有 13 个 frontmatter 字段和 6 个内置 agents 匹配。

---

## [2026-03-18 11:41 PM PKT] Claude Code v2.1.78

未检测到漂移 — 报告与官方文档完全同步。所有 13 个 frontmatter 字段和 6 个内置 agents 匹配。

---

## [2026-03-19 11:56 AM PKT] Claude Code v2.1.79

未检测到漂移 — 报告与官方文档完全同步。所有 13 个 frontmatter 字段和 6 个内置 agents 匹配。

---

## [2026-03-20 08:35 AM PKT] Claude Code v2.1.80

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Field | 在 Frontmatter Fields 表格中添加 `effort` 字段（string, optional — effort level override: `low`, `medium`, `high`, `max`） | ✅ COMPLETE (added between `background` and `isolation`, count updated 14→15) |

---

## [2026-03-21 09:07 PM PKT] Claude Code v2.1.81

未检测到漂移 — 报告与官方文档完全同步。所有 15 个 frontmatter 字段和 6 个内置 agents 匹配。

---

## [2026-03-23 09:49 PM PKT] Claude Code v2.1.81

未检测到漂移 — 报告与官方文档完全同步。所有 15 个 frontmatter 字段（14 official + 1 unofficial `color`）和 6 个内置 agents 匹配。

---

## [2026-03-25 08:07 PM PKT] Claude Code v2.1.83

未检测到漂移 — 报告与官方文档完全同步。所有 15 个 frontmatter 字段（14 official + 1 unofficial `color`）和 6 个内置 agents 匹配。

**关注项：** `initialPrompt` 在 v2.1.83 changelog 中添加但尚未出现在官方文档的 supported frontmatter fields 表格中。当它出现时，报告将需要更新。

---

## [2026-03-26 01:01 PM PKT] Claude Code v2.1.84

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Field | 在 Frontmatter Fields 表格中添加 `initialPrompt`（string, optional — 当 agent 通过 `--agent` 或 `agent` 设置作为 main session agent 运行时自动提交为第一个 user turn） | ✅ COMPLETE (added between `isolation` and `color`, count updated 15→16) |

---

## [2026-03-27 06:28 PM PKT] Claude Code v2.1.85

未检测到漂移 — 报告与官方文档完全同步。所有 16 个 frontmatter 字段（15 official + 1 unofficial `color`）和 6 个内置 agents 匹配。

---

## [2026-03-28 06:00 PM PKT] Claude Code v2.1.86

未检测到漂移 — 报告与官方文档完全同步。所有 16 个 frontmatter 字段（15 official + 1 unofficial `color`）和 6 个内置 agents 匹配。

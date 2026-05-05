---
description: 通过并行研究所有代理集合仓库来更新 AGENT COLLECTIONS 表格
---

<!-- 
 翻译来源：https://github.com/shanraisshan/claude-code-best-practice/blob/main/.claude/commands/workflows/agent-collections.md
 翻译时间：2026-05-06 02:00 CST
 翻译版本：v2.1.128
-->

# 工作流 — 代理集合（Agent Collections）

通过并行研究列出的仓库来更新 `README.md` 中的 AGENT COLLECTIONS 表格。启动研究代理，合并结果，报告变更，经批准后更新表格。

---

## 仓库列表

| # | 仓库 | 所有者 |
|---|------|-------|
| 1 | `msitarzewski/agency-agents` | msitarzewski |
| 2 | `VoltAgent/awesome-claude-code-subagents` | VoltAgent（精选 awesome-list） |

> 当发现新的代理集合仓库时，将它们添加到这里 AND Phase 1 的研究提示中。

---

## 表格格式

README 表格包含以下列：

```markdown
| Name | ★ | <img src="!/tags/a.svg" height="14"> |
```

- **名称（Name）**：`[短名称](github-url)` — 使用仓库的可识别短名称（如 `msitarzewski/agency-agents`、`awesome-claude-code-subagents`）。仅在名称不明确时使用完整的 `owner/repo`。
- **★（Stars）**：星标数四舍五入到 `k`（如 92k、19k、1.2k）。低于 1000 显示精确数字。
- **代理数量**：仅数字。对于 awesome-list 中代理是*链接*而非文件的情况，使用 `N+ (curated list)` 格式。

**排序顺序**：按星标数降序排列（最高的在前）。

---

## Phase 0：读取当前状态

读取以下文件：

1. `README.md` — `## 🤖 AGENT COLLECTIONS` 表格（记录当前星标数和代理数量）
2. `changelog/agent-collections/changelog.md` — 之前的变更日志条目（可能尚不存在 — 首次运行时创建）

---

## Phase 1：启动研究代理

**立即** spawn 一个 `development-workflows-research-agent` 覆盖所有仓库。（现有的研究代理是通用的 — 它可以统计任何仓库的 agents/skills/commands/stars。）

> 研究这些 Claude Code **代理集合（agent-collection）** 仓库。每个仓库主要是子代理定义文件的库（定义代理的 `.md` 文件），而非完整的工作流方法论。
>
> **仓库 1：msitarzewski/agency-agents** (https://github.com/msitarzewski/agency-agents) — agency 风格子代理集合
> **仓库 2：VoltAgent/awesome-claude-code-subagents** (https://github.com/VoltAgent/awesome-claude-code-subagents) — 精选 awesome-list（链接到外部子代理，并非所有代理都以文件形式存储在仓库中）
>
> 对于每个仓库，返回：
>
> 1. **星标数（Stars）** — 使用 GitHub API `https://api.github.com/repos/{owner}/{repo}`，读取 `stargazers_count`。四舍五入到 `k`。
> 2. **代理数量** — 通过 GitHub git tree API 统计子代理定义 `.md` 文件：
>    `https://api.github.com/repos/{owner}/{repo}/git/trees/HEAD?recursive=1` 并 grep 传统代理目录下的路径。
>    - 对于 `msitarzewski/agency-agents`：代理通常位于 `agents/`、`.claude/agents/` 或分类子目录下。统计看起来像子代理定义的 `.md` 文件（frontmatter 包含 `name:` 和 `description:`）。排除 README/CHANGELOG/LICENSE/docs。
>    - 对于 `VoltAgent/awesome-claude-code-subagents`：统计 README.md 中*列出*的代理（如链接到外部仓库的项目符号/表格行）。明确标记为"curated list, not files in repo"。
>    - 如果一个仓库既有精选索引又有自己的代理文件，报告两个数字并解释。
> 3. **显著变更** — 过去 30 天内有任何重大添加或移除吗？
>
> 为每个仓库返回结构化报告：
> ```
> REPO: msitarzewski/agency-agents
> STARS: <number>k (<exact>)
> AGENTS: <count> (<file pattern used, e.g., ".md files under agents/ via git tree">)
> NOTES: <anything unusual — flat layout vs categorized, README-only catalog, deprecated agents, curated-list disclaimer>
> CHANGES: <changes or "No significant changes">
> CONFIDENCE: <0-1>
> ```

---

## Phase 2：比较与报告

**等待代理完成。** 然后将研究结果与当前表格进行比较并展示：

```
Agent Collections — Update Report
══════════════════════════════════

Changes Found:
  <repo>: ★ <old>k → <new>k | agents <old>→<new>
  ...

No Changes:
  <repo>: ✓ (all values match)
  ...

Action Items:
#  | Type   | Action                              | Status
1  | Star   | Update <repo> ★ from Xk to Yk       | NEW/RECURRING
2  | Count  | Update <repo> agents from X to Y    | NEW/RECURRING
3  | Sort   | Move <repo> (rank changed)          | NEW/RECURRING
4  | Add    | New collection candidate: <repo>     | NEW
```

与之前的变更日志条目进行比较，将条目标记为 `NEW`、`RECURRING` 或 `RESOLVED`。

---

## Phase 2.5：追加到变更日志

**MANDATORY（强制）** — 在向用户展示之前始终执行。

读取 `changelog/agent-collections/changelog.md`，然后**追加**新条目。如果文件不存在，先创建 Status Legend 再添加第一个条目。

```markdown
---

## [<YYYY-MM-DD HH:MM AM/PM PKT>] Agent Collections Update

| # | Priority | Type | Action | Status |
|---|----------|------|--------|--------|
| 1 | HIGH/MED/LOW | <type> | <action> | <status> |
```

通过 `TZ=Asia/Karachi date "+%Y-%m-%d %I:%M %p PKT"` 获取时间。Status 必须是以下之一：
- `COMPLETE (reason)` | `INVALID (reason)` | `ON HOLD (reason)`

始终追加，切勿覆盖。

---

## Phase 2.6：更新 Last Updated 徽章

**MANDATORY（强制）** — 在 Phase 2.5 之后执行。

更新 `README.md` 第 4 行的徽章。通过 `TZ=Asia/Karachi date "+%b %d, %Y %-I:%M %p PKT"` 获取时间，URL 编码后替换徽章中的日期。不要将其记录为 action item。

---

## Phase 3：执行

询问用户：**(1) 执行全部** | **(2) 执行指定项** | **(3) 跳过**

执行时，编辑 `README.md` 中的 `## 🤖 AGENT COLLECTIONS` 表格：
- 更新每行的星标数和代理数量
- 保持排序顺序：星标数降序（最高的在前）
- 完全匹配现有格式（链接样式、星标的 k 后缀）

---

## 规则

1. **一个研究代理，所有仓库** — 单条消息，内部并行子获取
2. **绝不猜测** — 仅使用代理提供的数据
3. **不自动执行** — 先展示报告，等待批准
4. **始终追加变更日志**和**始终更新徽章** — 强制要求
5. **按星标数降序排列** — 星标最高的在前
6. **星标统一四舍五入** — `k` 后缀（92k、19k、1.2k）。低于 1000 显示精确数字
7. **Awesome-list 不同** — 对于链接到外部代理的仓库（VoltAgent），数量是"README 中列出的项目"，而非仓库中的文件；始终标注 `(curated list)`
8. **与之前的变更日志比较** — 标记条目为 NEW、RECURRING 或 RESOLVED
9. **复用 `development-workflows-research-agent`** — 不要创建新的代理

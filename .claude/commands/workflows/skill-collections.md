<!-- 
 翻译来源：https://github.com/shanraisshan/claude-code-best-practice/blob/main/.claude/commands/workflows/skill-collections.md
 翻译时间：2026-04-29 02:00 CST
 翻译版本：v1.0
-->

---
description: 并行调研全部 5 个 skill-collection 仓库，更新 README 中的 SKILL COLLECTIONS 表格
---

# 工作流 — Skill Collections

通过并行调研 5 个仓库来更新 `README.md` 中的 SKILL COLLECTIONS 表格。启动调研 Agent，合并结果，展示变更，经批准后更新表格。

---

## 5 个仓库

| # | 仓库 | 所有者 |
|---|------|-------|
| 1 | `anthropics/skills` | Anthropic（官方） |
| 2 | `wshobson/agents` | William Shobson |
| 3 | `mattpocock/skills` | Matt Pocock |
| 4 | `K-Dense-AI/scientific-agent-skills` | K-Dense-AI |
| 5 | `VoltAgent/awesome-agent-skills` | VoltAgent（精选列表） |

---

## 表格格式

README 表格包含以下列：

```markdown
| Name | ★ | <img src="!/tags/s.svg" height="14"> |
```

- **Name**: `[短名称](github-url)` — 使用仓库短名（如 `mattpocock/skills`，或所有者是项目本身时用 `skills`），除非有歧义否则不用完整 owner/repo
- **★**: 星标数，四舍五入到 `k`（如 125k、35k、1.2k）。低于 1000 显示精确数字
- **Skill 数量**: 只写数字。对于 awesome-list 类型（skill 是链接而非文件），使用 `N+ (curated list)` 格式

**排序**: 按星标数降序排列（最高在前）。

---

## 阶段 0：读取当前状态

读取以下文件：

1. `README.md` — `## 🧰 SKILL COLLECTIONS` 表格（记录当前星标和 skill 数）
2. `changelog/skill-collections/changelog.md` — 之前的变更日志（可能尚不存在）

---

## 阶段 1：启动调研 Agent

**立即** spawn 一个 `development-workflows-research-agent`，覆盖全部 5 个仓库。（现有的 research agent 是通用的 — 可以统计任何仓库的 skill/星标等数据。）

> 调研这 5 个 Claude Code **skill-collection** 仓库。它们主要是 `SKILL.md` 文件库，不是完整的工作流方法论。
>
> **仓库 1: anthropics/skills** (https://github.com/anthropics/skills) — Anthropic 官方 skill 仓库
> **仓库 2: wshobson/agents** (https://github.com/wshobson/agents) — 插件级 skill（skill 嵌套在领域插件下）
> **仓库 3: mattpocock/skills** (https://github.com/mattpocock/skills) — TypeScript 专注
> **仓库 4: K-Dense-AI/scientific-agent-skills** (https://github.com/K-Dense-AI/scientific-agent-skills) — 科学/研究领域
> **仓库 5: VoltAgent/awesome-agent-skills** (https://github.com/VoltAgent/awesome-agent-skills) — 精选 awesome-list（链接到外部 skill，仓库内没有 SKILL.md 文件）
>
> 对**每个**仓库，返回：
>
> 1. **Stars** — 使用 GitHub API `https://api.github.com/repos/{owner}/{repo}`，读取 `stargazers_count`。四舍五入到 `k`。
> 2. **Skill 数量** — 通过 GitHub git tree API 统计 `SKILL.md` 文件数：
>    `https://api.github.com/repos/{owner}/{repo}/git/trees/HEAD?recursive=1` 并 grep 路径中的 `SKILL.md`。
>    - 对于 `wshobson/agents`：skill 嵌套在 `plugins/<domain>/skills/` 中 — 统计所有插件中的 SKILL.md。
>    - 对于 `VoltAgent/awesome-agent-skills`：统计 README.md 中*列出*的 skill（如列表项 / 表格行）。明确标注为"精选列表，非文件"。
>    - 对于 `K-Dense-AI/scientific-agent-skills`：`skills/` 下的子目录可能使用 SKILL.md 或 `.md`；统计仓库实际使用的格式，并报告使用了哪种。
>    - 对于 `anthropics/skills`：skill 存放在 `skills/` 下的子目录中，内有 `SKILL.md`。
>    - 对于 `mattpocock/skills`：只统计**活跃** skill，不含已废弃的（如果明显则两个数字都报告）。
> 3. **显著变更** — 过去 30 天有重大增减吗？
>
> 按以下格式返回每个仓库的结构化报告：
> ```
> REPO: anthropics/skills
> STARS: <number>k (<exact>)
> SKILLS: <count> (<file pattern used, e.g., "SKILL.md files via git tree">)
> NOTES: <anything unusual — flat .md vs SKILL.md, deprecated skills, language variants, curated-list disclaimer>
> CHANGES: <changes or "No significant changes">
> CONFIDENCE: <0-1>
> ```

---

## 阶段 2：对比与报告

**等待 Agent 完成。** 然后将调研结果与当前表格对比并展示：

```
Skill Collections — 更新报告
══════════════════════════════════

发现的变更：
  <repo>: ★ <old>k → <new>k | skills <old>→<new>
  ...

无变更：
  <repo>: ✓ (所有值匹配)
  ...

待执行操作：
#  | 类型   | 操作                              | 状态
1  | Star   | 更新 <repo> ★ 从 Xk 到 Yk       | NEW/RECURRING
2  | Count  | 更新 <repo> skills 从 X 到 Y    | NEW/RECURRING
3  | Sort   | 移动 <repo>（排名变更）          | NEW/RECURRING
4  | Add    | 新集合候选：<repo>                | NEW
```

与之前的 changelog 条目对比，将条目标记为 `NEW`、`RECURRING` 或 `RESOLVED`。

---

## 阶段 2.5：追加到变更日志

**强制** — 在向用户展示前必须执行。

读取 `changelog/skill-collections/changelog.md`，然后**追加**一条新记录。如果文件不存在，先创建状态图例再写第一条记录。

```markdown
---

## [<YYYY-MM-DD HH:MM AM/PM PKT>] Skill Collections Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH/MED/LOW | <type> | <action> | <status> |
```

通过 `TZ=Asia/Karachi date "+%Y-%m-%d %I:%M %p PKT"` 获取时间。状态必须是以下之一：
- `COMPLETE (原因)` | `INVALID (原因)` | `ON HOLD (原因)`

始终追加，绝不覆盖。

---

## 阶段 2.6：更新 Last Updated 徽章

**强制** — 在阶段 2.5 之后执行。

更新 `README.md` 第 4 行的徽章。通过 `TZ=Asia/Karachi date "+%b %d, %Y %-I:%M %p PKT"` 获取时间，URL 编码后替换徽章中的日期。不要将此记录为 action item。

---

## 阶段 3：执行

询问用户：**(1) 执行全部** | **(2) 执行指定项** | **(3) 跳过**

执行时，编辑 `README.md` 中的 `## 🧰 SKILL COLLECTIONS` 表格：
- 更新每行的星标和 skill 数
- 保持排序：星标降序（最高在前）
- 精确匹配已有格式（链接样式、星标的 k 后缀）

---

## 规则

1. **一个调研 Agent，5 个仓库** — 单条消息，内部并行子获取
2. **绝不猜测** — 只使用 Agent 返回的数据
3. **不要自动执行** — 先展示报告，等待批准
4. **始终追加变更日志** 且 **始终更新徽章** — 强制
5. **按星标降序排序** — 星标最高在前
6. **星标舍入一致** — `k` 后缀（125k、35k、1.2k）。低于 1000 显示精确值
7. **Awesome-list 不同** — 对于链接到外部 skill 的仓库（VoltAgent），计数是"README 中列出的条目"而非仓库文件；始终标注 `(curated list)`
8. **与之前的变更日志对比** — 将条目标记为 NEW、RECURRING 或 RESOLVED
9. **复用 `development-workflows-research-agent`** — 不要创建新的 Agent

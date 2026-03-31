<!-- 翻译自：https://github.com/shanraisshan/claude-code-best-practice/blob/main/changelog/best-practice/concepts/changelog.md -->

# Changelog — README CONCEPTS Section（变更日志 — README CONCEPTS 部分）

跟踪 README CONCEPTS 表格与官方 Claude Code 文档之间的漂移。

## 状态图例

| 状态 | 含义 |
|--------|---------|
| ✅ `COMPLETE (reason)` | 操作已成功执行并解决 |
| ❌ `INVALID (reason)` | 发现不正确、不适用或有意为之 |
| ✋ `ON HOLD (reason)` | 操作已推迟 — 等待外部依赖或用户决策 |

---

## [2026-03-02 11:14 AM PKT] Claude Code v2.1.63

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Broken URL | 修复 Permissions URL 从 `/iam` 到 `/permissions` | ✅ COMPLETE (URL updated to /permissions) |
| 2 | HIGH | Missing Concept | 在 CONCEPTS 表格中添加 Agent Teams 行 | ✅ COMPLETE (row added with ~\/\.claude\/teams\/ location) |
| 3 | HIGH | Missing Concept | 在 CONCEPTS 表格中添加 Keybindings 行 | ✅ COMPLETE (row added with ~\/\.claude\/keybindings\.json location) |
| 4 | HIGH | Missing Concept | 在 CONCEPTS 表格中添加 Model Configuration 行 | ✅ COMPLETE (row added with \.claude\/settings\.json location) |
| 5 | HIGH | Missing Concept | 在 CONCEPTS 表格中添加 Auto Memory 行 | ✅ COMPLETE (row added with ~\/\.claude\/projects\/<project>\/memory\/ location) |
| 6 | HIGH | Stale Anchor | 修复 Rules URL anchor 从 `#modular-rules-with-clauderules` 到 `#organize-rules-with-clauderules` | ✅ COMPLETE (anchor updated) |
| 7 | MED | Missing Concept | 在 CONCEPTS 表格中添加 Checkpointing 行 | ✅ COMPLETE (row added with automatic git-based location) |
| 8 | MED | Missing Concept | 在 CONCEPTS 表格中添加 Status Line 行 | ✅ COMPLETE (row added with ~\/\.claude\/settings\.json location) |
| 9 | MED | Missing Concept | 在 CONCEPTS 表格中添加 Remote Control 行 | ✅ COMPLETE (row added with CLI \/ claude\.ai location) |
| 10 | MED | Missing Concept | 在 CONCEPTS 表格中添加 Fast Mode 行 | ✅ COMPLETE (row added with \.claude\/settings\.json location) |
| 11 | MED | Missing Concept | 在 CONCEPTS 表格中添加 Headless Mode 行 | ✅ COMPLETE (row added with CLI flag -p location) |
| 12 | LOW | Changed Description | 更新 Memory 描述以提及 auto memory | ✅ COMPLETE (description and location updated) |
| 13 | LOW | Changed Location | 更新 MCP Servers 位置以包含 `.mcp.json` | ✅ COMPLETE (location updated to include .mcp.json) |
| 14 | LOW | Missing Badge | 为 Hooks 行添加 Implemented 徽章 | ✅ COMPLETE (Implemented badge added linking to .claude/hooks/) |

---

## [2026-03-02 11:57 AM PKT] Claude Code v2.1.63

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Table Consolidation | 将 CONCEPTS 表格从 22 行整合为 10 行 — 将相关概念折叠为内联文档链接 | ✅ COMPLETE (22 → 10 rows) |
| 2 | MED | Merged Concept | 将 Marketplaces 折叠到 Plugins 行作为内联链接 | ✅ COMPLETE (linked to /discover-plugins) |
| 3 | MED | Merged Concept | 将 Agent Teams 折叠到 Sub-Agents 行作为内联链接 | ✅ COMPLETE (linked to /agent-teams) |
| 4 | MED | Merged Concept | 将 Permissions, Model Config, Output Styles, Sandboxing, Keybindings, Status Line, Fast Mode 折叠到 Settings 行作为内联链接 | ✅ COMPLETE (7 concepts folded with doc links) |
| 5 | MED | Merged Concept | 将 Auto Memory 和 Rules 折叠到 Memory 行作为内联链接 | ✅ COMPLETE (linked to /memory and /memory#organize-rules-with-clauderules) |
| 6 | MED | Merged Concept | 将 Headless Mode 折叠到 Remote Control 行作为内联链接 | ✅ COMPLETE (linked to /headless) |
| 7 | LOW | Reorder | 按逻辑分组重新排序：building blocks → extension → config → context → runtime | ✅ COMPLETE (grouped by concern, not chronology) |

---

## [2026-03-07 08:40 AM PKT] Claude Code v2.1.71

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Broken URL | 修复 TIPS 中的 `context-management` → `interactive-mode`（第 112, 115, 135 行） | ✅ COMPLETE (3 occurrences replaced with interactive-mode) |
| 2 | HIGH | Broken URL | 修复 TIPS 中的 `model-configuration` → `model-config`（第 115, 116, 135 行） | ✅ COMPLETE (3 occurrences replaced with model-config) |
| 3 | HIGH | Broken URL | 修复 TIPS 中的 `usage-billing` → `costs`（第 115 行） | ✅ COMPLETE (replaced with costs) |
| 4 | HIGH | Broken URL | 移除 STARTUPS 中的 `cowork` URL（第 167 行）— 页面不存在 | ✅ COMPLETE (hyperlink removed, plain text kept) |
| 5 | HIGH | Missing Concept | 在 CONCEPTS 和 Hot 部分添加 Scheduled Tasks 行（`/loop`, cron tools） | ✅ COMPLETE (added by user to both tables + /loop tip + Boris tweet) |
| 6 | MED | Changed Location | 更新 Agent Teams 位置从 `.claude/agents/<name>.md` 到 `built-in (env var)` | ✅ COMPLETE (location updated to built-in env var) |

---

## [2026-03-10 01:18 PM PKT] Claude Code v2.1.72

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Broken URL | 修复 CONCEPTS 表格中的 Commands URL 从 `/slash-commands` 到 `/skills`（第 24 行）— `/slash-commands` 提供 Skills 页面内容；文档说 "commands merged into skills" | ❌ INVALID (URL still resolves; user chose to keep as-is) |
| 2 | HIGH | Broken URL | 修复 TIPS 部分中的 Commands URL 从 `/slash-commands` 到 `/skills`（第 108 行）— 相同的过时 URL | ❌ INVALID (URL still resolves; user chose to keep as-is) |
| 3 | MED | Missing Inline Link | 添加 Interactive Mode（`/interactive-mode`）作为内联链接到 CLI Startup Flags 行 — 涵盖 /compact, /clear, /context, /extra-usage | ✅ COMPLETE (inline link added to CLI Startup Flags description) |
| 4 | MED | Missing Inline Link | 添加 Costs（`/costs`）作为内联链接到 Settings 行 — 涵盖 /usage, billing, pay-as-you-go | ❌ INVALID (user chose to skip) |
| 5 | LOW | Missing Concept | 考虑在 Best Practices 中添加 IDE Integrations 行（VS Code, JetBrains, Desktop App, Web）或内联链接 | ❌ INVALID (user chose to skip — platform surfaces, not configuration concepts) |
| 6 | HIGH | Missing Concept | 在 Hot 表格中添加 Code Review 行 — 多 Agent PR 分析（research preview, Teams & Enterprise） | ✅ COMPLETE (row added as first Hot entry with blog link and best practice tweet) |
| 7 | MED | New Badge | 创建 `!/tags/beta.svg` 徽章（黄色，38x20px）并添加到 Hot 表格中的 Code Review 和 Agent Teams | ✅ COMPLETE (beta.svg created; added to Code Review and Agent Teams rows) |
| 8 | MED | Reorder | 按发布日期排序 Hot 表格（最近优先）：Code Review → Scheduled Tasks → Voice Mode → Agent Teams → Remote Control → Git Worktrees → Ralph Wiggum | ✅ COMPLETE (Voice Mode and Agent Teams swapped to match chronological order) |

---

## [2026-03-12 12:22 PM PKT] Claude Code v2.1.74

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Broken URL | 修复 CONCEPTS 表格中的 Commands URL 从 `/slash-commands` 到 `/skills`（第 24 行）— `/slash-commands` 重定向到 `/skills` 页面 | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | LOW | Verification | 所有外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all 20+ URLs return valid pages) |
| 3 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all badge targets exist on filesystem) |
| 4 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在目标页面上已验证 | ✅ COMPLETE (heading exists on /memory page) |
| 5 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 | ✅ COMPLETE (no description drift detected) |

---

## [2026-03-15 12:48 PM PKT] Claude Code v2.1.76

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | MED | Missing Badges | Remote Control (Hot) 零徽章 — Hot 中唯一没有任何 BP 或 Impl 徽章的项目 | ✅ COMPLETE (BP badge added linking to official docs page) |
| 3 | LOW | Naming | README 中的 "Sub-Agents" 与官方文档中的 "subagents"（一个词）— 表面不一致 | ✅ COMPLETE (renamed to "Subagents" in CONCEPTS table) |
| 4 | LOW | Verification | 所有 27 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages) |
| 5 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all badge targets exist on filesystem) |
| 6 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面上确认 | ✅ COMPLETE (section heading exists) |
| 7 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (descriptions accurate for all 13 CONCEPTS + 9 Hot rows) |

---

## [2026-03-17 12:46 PM PKT] Claude Code v2.1.77

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | HIGH | Changed Description | Hooks 描述说 "Deterministic scripts" 但 hooks 现在包括 4 种类型：command, HTTP, prompt, and agent — 只有 command hooks 是 deterministic | ✅ COMPLETE (updated to "User-defined handlers (scripts, HTTP, prompts, agents)" in CONCEPTS table) |
| 3 | MED | Missing Concept | Desktop App 在 `/desktop` 有专用文档页面 — 不在 CONCEPTS 或 Hot 表格中 | ❌ INVALID (user chose to skip — Desktop is a platform surface, not a configuration concept) |
| 4 | MED | Changed URL | Hooks 文档现在分为 Guide（`/hooks-guide`）和 Reference（`/hooks`）— CONCEPTS 仅链接到 Reference | ✅ COMPLETE (Guide link added as inline link in Hooks row description) |
| 5 | LOW | Verification | 所有 28 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 6 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20 badge targets exist on filesystem) |
| 7 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 8 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 | ✅ COMPLETE (Hooks description drift detected — see #2) |

---

## [2026-03-18 11:43 PM PKT] Claude Code v2.1.78

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | HIGH | Changed URL+Name | Hot 表格中的 Voice Mode 链接到 tweet 而非官方文档 `/voice-dictation`；官方名称是 "Voice Dictation" | ✅ COMPLETE (renamed to "Voice Dictation", linked to /voice-dictation, description updated; BP badge kept linking to tweet; also updated in STARTUPS table) |
| 3 | LOW | Verification | 所有 29 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 4 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 5 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 6 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |

---

## [2026-03-19 11:59 AM PKT] Claude Code v2.1.79

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | LOW | Verification | 所有 30 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 3 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 4 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 5 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |

---

## [2026-03-20 08:38 AM PKT] Claude Code v2.1.80

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Missing Concept | 在 Hot 表格中添加 Channels 行 — 从 Telegram/Discord/webhooks 推送事件到运行中的会话（research preview, v2.1.80） | ✅ COMPLETE (row added as first Hot entry with beta badge and Reference link) |
| 2 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 3 | MED | Missing Deep Link | Git Worktrees URL 应 anchor 到 `#run-parallel-claude-code-sessions-with-git-worktrees` | ✅ COMPLETE (anchor added to Git Worktrees URL in Hot table) |
| 4 | LOW | Missing Inline Link | Plugins 行可以添加 `[Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)` 子链接 | ✅ COMPLETE (Create Marketplaces inline link added to Plugins row) |
| 5 | LOW | Verification | 所有 31 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 6 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 7 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 8 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |

---

## [2026-03-21 09:12 PM PKT] Claude Code v2.1.81

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | LOW | Verification | 所有 32 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 3 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 4 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 5 | LOW | Verification | Git Worktrees anchor `#run-parallel-claude-code-sessions-with-git-worktrees` 在 /common-workflows 页面确认 | ✅ COMPLETE (section heading exists) |
| 6 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |

---

## [2026-03-23 09:53 PM PKT] Claude Code v2.1.81

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | LOW | Verification | 所有 33 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 3 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 4 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 5 | LOW | Verification | Git Worktrees anchor `#run-parallel-claude-code-sessions-with-git-worktrees` 在 /common-workflows 页面确认 | ✅ COMPLETE (section heading exists) |
| 6 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |

---

## [2026-03-25 08:12 PM PKT] Claude Code v2.1.83

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | MED | Changed URL | Simplify & Batch 主链接指向 tweet 而非官方文档 `/skills#bundled-skills` — 现在是官方 bundled skills | ✅ COMPLETE (primary link updated to /skills#bundled-skills; BP badge kept linking to Boris's tweet) |
| 3 | LOW | Verification | 所有 34 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 4 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 5 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 6 | LOW | Verification | Git Worktrees anchor `#run-parallel-claude-code-sessions-with-git-worktrees` 在 /common-workflows 页面确认 | ✅ COMPLETE (section heading exists) |
| 7 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |
| 8 | HIGH | Missing Concept | 在 Hot 表格中添加 Auto Mode 行 — 后台安全分类器替换权限提示（research preview, Team/Enterprise） | ✅ COMPLETE (row added as first Hot entry with beta badge, BP badge linking to @claudeai tweet, and blog link) |

---

## [2026-03-26 01:05 PM PKT] Claude Code v2.1.84

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | MED | Missing Concept | 在 Hot 表格中添加 Slack integration — 在 Slack 中提及 @Claude 以将编码任务路由到 Claude Code web sessions | ✅ COMPLETE (row added after Channels with @Claude location and web session description) |
| 3 | MED | Missing Concept | 在 Hot 表格中添加 GitHub Actions / CI-CD — 在 CI/CD pipelines 中自动化 PR reviews, issue triage, and code generation | ✅ COMPLETE (row added after Code Review with .github/workflows/ location and GitLab CI/CD inline link) |
| 4 | LOW | Verification | 所有 35 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 5 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 6 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 7 | LOW | Verification | Git Worktrees anchor `#run-parallel-claude-code-sessions-with-git-worktrees` 在 /common-workflows 页面确认 | ✅ COMPLETE (section heading exists) |
| 8 | LOW | Verification | Auto Mode anchor `#eliminate-prompts-with-auto-mode` 在 /permission-modes 页面确认 | ✅ COMPLETE (section heading exists) |
| 9 | LOW | Verification | Bundled Skills anchor `#bundled-skills` 在 /skills 页面确认 | ✅ COMPLETE (section heading exists) |
| 10 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |

---

## [2026-03-27 06:37 PM PKT] Claude Code v2.1.85

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | MED | Missing Concept | 在 Hot 表格中添加 Chrome integration — 通过 Claude in Chrome extension 进行浏览器自动化（beta, 专用文档 `/chrome`） | ✅ COMPLETE (row added after GitHub Actions with --chrome location and beta badge) |
| 3 | LOW | Verification | 所有 36 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 4 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 5 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading exists) |
| 6 | LOW | Verification | Git Worktrees anchor `#run-parallel-claude-code-sessions-with-git-worktrees` 在 /common-workflows 页面确认 | ✅ COMPLETE (section heading exists) |
| 7 | LOW | Verification | Auto Mode anchor `#eliminate-prompts-with-auto-mode` 在 /permission-modes 页面确认 | ✅ COMPLETE (section heading exists) |
| 8 | LOW | Verification | Bundled Skills anchor `#bundled-skills` 在 /skills 页面确认 | ✅ COMPLETE (section heading exists) |
| 9 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate) |

---

## [2026-03-28 06:04 PM PKT] Claude Code v2.1.86

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Stale URL | Commands URL `/slash-commands` 提供 Skills 页面 — 文档说 "commands merged into skills" | ❌ INVALID (RECURRING from 2026-03-10; URL still resolves; user chose to keep as-is) |
| 2 | MED | Missing Badge | Hot 表格中的 Chrome 行没有 BP 徽章 — 报告存在于 `reports/claude-in-chrome-v-chrome-devtools-mcp.md` | ✅ COMPLETE (BP badge added linking to reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| 3 | LOW | Changed Description | Plugins 描述缺少 LSP servers — 官方文档列出 `.lsp.json` 作为 plugin 组件 | ✅ COMPLETE (added "and LSP servers" to Plugins description) |
| 4 | LOW | Verification | 所有 37 个外部文档 URLs 已验证 — 未发现损坏链接 | ✅ COMPLETE (all URLs return valid pages including /slash-commands redirect) |
| 5 | LOW | Verification | 所有本地徽章文件路径已验证 — 无缺失文件 | ✅ COMPLETE (all 20+ badge targets exist on filesystem) |
| 6 | LOW | Verification | Memory anchor `#organize-rules-with-clauderules` 在 /memory 页面确认 | ✅ COMPLETE (section heading `.claude/rules/` exists) |
| 7 | LOW | Verification | Git Worktrees anchor `#run-parallel-claude-code-sessions-with-git-worktrees` 在 /common-workflows 页面确认 | ✅ COMPLETE (section heading exists) |
| 8 | LOW | Verification | Auto Mode anchor `#eliminate-prompts-with-auto-mode` 在 /permission-modes 页面确认 | ✅ COMPLETE (section heading exists) |
| 9 | LOW | Verification | Bundled Skills anchor `#bundled-skills` 在 /skills 页面确认 | ✅ COMPLETE (section heading exists) |
| 10 | LOW | Verification | 所有 CONCEPTS 描述已与官方文档核对 — 未检测到漂移 | ✅ COMPLETE (all descriptions accurate except Plugins LSP note — see #3) |

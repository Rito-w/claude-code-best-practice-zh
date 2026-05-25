# Verification Checklist — README CONCEPTS Section

Rules for verifying CONCEPTS table accuracy. Each rule is checked during every workflow run.

## Rules

### 1. External URL Liveness
- **Category**: URL Accuracy
- **What to check**: Every external URL in the CONCEPTS table (docs links) returns a valid page
- **Depth**: Fetch each URL and confirm it loads the expected page (not a redirect to wrong page)
- **Source to compare against**: `https://code.claude.com/docs/llms.txt` for canonical URL list
- **Date added**: 2026-03-02
- **Origin**: Permissions URL `/iam` was found to redirect to Authentication page instead of Permissions

### 2. Anchor Fragment Validity
- **Category**: URL Accuracy
- **What to check**: Any URL with an anchor fragment (`#section-name`) matches an actual heading on the target page
- **Depth**: Fetch the page and verify the heading exists with the expected anchor
- **Source to compare against**: Fetched page content
- **Date added**: 2026-03-02
- **Origin**: Rules anchor `#modular-rules-with-clauderules` was stale; section renamed to `#organize-rules-with-clauderules`

### 3. Missing Docs Pages
- **Category**: Missing Concepts
- **What to check**: Every page in the official docs index (`llms.txt`) that represents a user-facing feature has a corresponding row in the CONCEPTS table
- **Depth**: Compare full docs index against CONCEPTS table entries
- **Source to compare against**: `https://code.claude.com/docs/llms.txt`
- **Date added**: 2026-03-02
- **Origin**: Multiple missing concepts found (Agent Teams, Keybindings, Model Configuration, etc.)

### 4. Local Badge Link Validity
- **Category**: Badge Accuracy
- **What to check**: Every badge target path in the CONCEPTS table (`best-practice/*.md`, `implementation/*.md`, `.claude/*/`) points to a file or directory that exists
- **Depth**: Use Read/Glob to verify file existence
- **Source to compare against**: Local filesystem
- **Date added**: 2026-03-02
- **Origin**: Initial checklist creation

### 5. Description Currency
- **Category**: Description Accuracy
- **What to check**: Each concept's description accurately reflects the current official docs description
- **Depth**: Compare README description against the official page's meta description or first paragraph
- **Source to compare against**: Official docs page content
- **Date added**: 2026-03-02
- **Origin**: Memory description missing auto memory; MCP Servers location missing `.mcp.json`

### 6. TIPS Section URL Consistency
- **Category**: URL Accuracy
- **What to check**: When a URL is updated in the CONCEPTS or Hot table, also check the TIPS section for the same stale URL
- **Depth**: Search TIPS section (lines 125–267) for every URL that was flagged in the CONCEPTS/Hot table
- **Source to compare against**: llms.txt sitemap + CONCEPTS table URLs
- **Date added**: 2026-04-16
- **Origin**: `web-scheduled-tasks` URL was fixed in Hot table (2026-04-14) but the same stale URL persisted in TIPS (line 223)

### 7. Beta Badge Currency
- **Category**: Badge Accuracy
- **What to check**: Concepts marked `![beta](!/tags/beta.svg)` in the Hot table should still be flagged as beta / research preview in their official docs page (header banner, "research preview" copy, or env-var gating)
- **Depth**: Fetch each upstream page and check for explicit beta/preview/experimental wording; if absent, the README badge may be stale
- **Source to compare against**: Official docs page banner text + GA-marker copy
- **Date added**: 2026-04-26
- **Origin**: Workflow-concepts-agent flagged Routines, No Flicker Mode, Computer Use, and Code Review as carrying README beta badges while their docs pages read as GA (confidence 0.6) — this drift type isn't covered by the existing description-currency rule

### 8. Location Column Factual Accuracy
- **Category**: Description Accuracy
- **What to check**: Each concept's **Location** column (column 2) value must factually match the mechanism described in the official docs — not merely that the path/command/flag exists, but that the characterization is correct (e.g. how a feature stores state, what it tracks, where config lives)
- **Depth**: For any Location value that makes a mechanism claim (e.g. "git-based", "automatic", "built-in (env var)", "cloud backend"), fetch the official page and confirm the claim against the docs' own description of the mechanism
- **Source to compare against**: Official docs page body (mechanism/"how it works" sections)
- **Date added**: 2026-05-21
- **Origin**: Checkpointing Location read `automatic (git-based)` for every prior run, but `/en/checkpointing` explicitly states checkpoints track file-editing-tool changes (not git, not bash) and are "not a replacement for version control" — *"checkpoints as 'local undo' and Git as 'permanent history'."* Existing rule #5 (Description Currency) only inspects the Description column, so a factual error in the Location column went unchecked indefinitely

### 9. 捆绑技能/命令重命名跟踪
- **类别**: 描述准确性（Location 列）
- **检查内容**: 在 CONCEPTS/Hot **Location** 列中命名的任何斜杠命令或捆绑技能（如 `/simplify`、`/code-review`、`/loop`、`/batch`、`/goal`、`/ultrareview`）仍以该确切名称存在 — 即未被重命名或在近期版本中移除
- **深度**: 扫描最近 N 个版本的 CHANGELOG 中"renamed"、"removed"或"deprecated"命令/技能事件；交叉检查当前 `/en/commands` 参考和 `/en/skills#bundled-skills` 列表以获取实际当前命令名称
- **对比来源**: `https://raw.githubusercontent.com/anthropics/claude-code/main/CHANGELOG.md` + `https://code.claude.com/docs/en/commands` + `https://code.claude.com/docs/en/skills#bundled-skills`
- **添加日期**: 2026-05-25
- **来源**: `/simplify` 在 v2.1.147 中重命名为 `/code-review` — 这是一个**仅更新日志事件**，文档页面列表检查（规则 #1/#3）和 claude-code-guide 95 概念扫描均未发现；只有原始 CHANGELOG 读取捕获到它

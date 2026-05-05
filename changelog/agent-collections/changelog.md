---
description: Update the AGENT COLLECTIONS table by researching all agent-collection repos in parallel
---

<!-- 
 翻译来源：https://github.com/shanraisshan/claude-code-best-practice/blob/main/changelog/agent-collections/changelog.md
 翻译时间：2026-05-06 02:00 CST
 翻译版本：v2.1.128
-->

# 代理集合 — 变更日志（Agent Collections — Changelog）

跟踪 `README.md` 中 AGENT COLLECTIONS 表格的更新。

## 状态图例

- `COMPLETE (reason)` — 操作项已成功执行
- `INVALID (reason)` — 操作项被判定为不必要或不正确
- `ON HOLD (reason)` — 操作项暂缓执行

---

## [2026-05-05 09:26 PM PKT] 代理集合更新

| # | 优先级 | 类型 | 操作 | 状态 |
|---|--------|------|------|------|
| 1 | MED | 星标 | 更新 msitarzewski/agency-agents ★ 从 92k 到 93k | COMPLETE (verified via GitHub API: 93,374) |
| 2 | MED | 数量 | 更新 msitarzewski/agency-agents agents 从 206 到 197 | COMPLETE (recursive tree count, agent .md files across 15 categories) |
| 3 | LOW | 星标 | VoltAgent/awesome-claude-code-subagents ★ 未变更 (19k = 19,137) | INVALID (no change required) |
| 4 | MED | 数量 | 更新 VoltAgent/awesome-claude-code-subagents agents 从 148 到 144 | COMPLETE (recursive tree count under categories/, excluding tools/) |
| 5 | LOW | 排序 | 验证排序顺序（星标降序） | COMPLETE (msitarzewski 93k > VoltAgent 19k — order preserved) |
| 6 | MED | 规则 | 确认表格入选的 10k+ 星标阈值 | COMPLETE (user confirmed; both listed repos pass — msitarzewski 93k, VoltAgent 19k; saved as feedback memory for future runs) |

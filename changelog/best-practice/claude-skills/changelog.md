
# Skills Report Changelog（Skills 报告变更日志）

**状态图例：**

| 状态 | 含义 |
|--------|---------|
| ✅ `COMPLETE (reason)` | 操作已成功执行并解决 |
| ❌ `INVALID (reason)` | 发现不正确、不适用或有意为之 |
| ✋ `ON HOLD (reason)` | 操作已推迟 — 等待外部依赖或用户决策 |

---

## [2026-03-13 04:22 PM PKT] Claude Code v2.1.74

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | MED | Extra Bundled Skill | `keybindings-help` 在本地报告中但不在官方文档 bundled skills 列表中 — 调查是否移除或保留 | ✅ COMPLETE (removed from bundled skills table — it is a local custom skill in this repo, not an official bundled skill; `/keybindings` is a built-in CLI command) |

---

## [2026-03-15 12:49 PM PKT] Claude Code v2.1.76

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | LOW | Field Accuracy | `name` 字段 Required 列在本地报告中显示 "Recommended" 但官方文档现在列为 "No"（可选）— 更新以匹配 | ✅ COMPLETE (updated `name` Required from "Recommended" to "No" to match official docs) |

---

## [2026-03-17 12:42 PM PKT] Claude Code v2.1.77

无漂移检测 — frontmatter 字段（10）和 bundled skills（5）与官方文档完全同步。

---

## [2026-03-18 11:38 PM PKT] Claude Code v2.1.78

无漂移检测 — frontmatter 字段（10）和 bundled skills（5）与官方文档完全同步。

---

## [2026-03-19 11:54 AM PKT] Claude Code v2.1.79

无漂移检测 — frontmatter 字段（10）和 bundled skills（5）与官方文档完全同步。

---

## [2026-03-20 08:32 AM PKT] Claude Code v2.1.80

无漂移检测 — frontmatter 字段（10）和 bundled skills（5）与官方文档完全同步。

---

## [2026-03-21 09:07 PM PKT] Claude Code v2.1.81

无漂移检测 — frontmatter 字段（11）和 bundled skills（5）与官方文档完全同步。

---

## [2026-03-23 09:48 PM PKT] Claude Code v2.1.81

无漂移检测 — frontmatter 字段（11）和 bundled skills（5）与官方文档完全同步。

---

## [2026-03-25 08:06 PM PKT] Claude Code v2.1.83

无漂移检测 — frontmatter 字段（11）和 bundled skills（5）与官方文档完全同步。

---

## [2026-03-26 12:59 PM PKT] Claude Code v2.1.84

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Field | 在 frontmatter 表格中添加 `shell` 字段 — 接受 `bash`（默认）或 `powershell`，控制 skill 内容中 `!command` 块的 shell | ✅ COMPLETE (added to frontmatter table, count updated 11→12) |

---

## [2026-03-27 06:25 PM PKT] Claude Code v2.1.85

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Field | 在 frontmatter 表格中添加 `paths` 字段 — 接受 glob patterns（字符串或 YAML 列表），限制 skill 何时自动激活 | ✅ COMPLETE (added to frontmatter table, count updated 12→13) |

---

## [2026-03-28 05:59 PM PKT] Claude Code v2.1.86

无漂移检测 — frontmatter 字段（13）和 bundled skills（5）与官方文档完全同步。

<!-- 翻译自：https://github.com/shanraisshan/claude-code-best-practice/blob/main/changelog/best-practice/claude-commands/changelog.md -->

# Commands Report — Changelog History（Commands 报告 — 变更日志历史）

## 状态图例

| 状态 | 含义 |
|--------|---------|
| ✅ `COMPLETE (reason)` | 操作已成功执行并解决 |
| ❌ `INVALID (reason)` | 发现不正确、不适用或有意为之 |
| ✋ `ON HOLD (reason)` | 操作已推迟 — 等待外部依赖或用户决策 |

---

## [2026-03-13 04:23 PM PKT] Claude Code v2.1.74

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Field | 在 frontmatter 表格中添加 `name` — Skill 的显示名称 | ❌ INVALID (skill-only field, not applicable to commands frontmatter) |
| 2 | HIGH | New Field | 在 frontmatter 表格中添加 `disable-model-invocation` — 防止自动加载 | ❌ INVALID (skill-only field, not applicable to commands frontmatter) |
| 3 | HIGH | New Field | 在 frontmatter 表格中添加 `user-invocable` — 从 `/` 菜单隐藏 | ❌ INVALID (skill-only field, not applicable to commands frontmatter) |
| 4 | HIGH | New Field | 在 frontmatter 表格中添加 `context` — 在 subagent context 中运行的 fork | ❌ INVALID (skill-only field, not applicable to commands frontmatter) |
| 5 | HIGH | New Field | 在 frontmatter 表格中添加 `agent` — context: fork 的 subagent 类型 | ❌ INVALID (skill-only field, not applicable to commands frontmatter) |
| 6 | HIGH | New Field | 在 frontmatter 表格中添加 `hooks` —  scoped to skill 的生命周期 hooks | ❌ INVALID (skill-only field, not applicable to commands frontmatter) |
| 7 | HIGH | New Command | 添加 `/btw <question>` — 快速询问侧边问题而不添加到对话中 | ✅ COMPLETE (added as #53 in Session tag) |
| 8 | HIGH | New Command | 添加 `/hooks` — 管理工具事件的 hook 配置 | ✅ COMPLETE (added as #30 in Extensions tag) |
| 9 | HIGH | New Command | 添加 `/insights` — 生成会话分析报告 | ✅ COMPLETE (added as #17 in Context tag) |
| 10 | HIGH | New Command | 添加 `/plugin` — 管理 Claude Code plugins | ✅ COMPLETE (added as #33 in Extensions tag) |
| 11 | HIGH | New Command | 添加 `/skills` — 列出可用 skills | ✅ COMPLETE (added as #35 in Extensions tag) |
| 12 | HIGH | New Command | 添加 `/upgrade` — 打开升级页面以切换计划层级 | ✅ COMPLETE (added as #3 in Auth tag) |
| 13 | HIGH | Removed Command | 移除 `/output-style` — 在 v2.1.73 中已弃用，使用 `/config` 替代 | ✅ COMPLETE (removed from Config tag) |
| 14 | HIGH | Removed Command | 移除 `/bug` 行 — 现在作为别名列在 `/feedback` 下 | ✅ COMPLETE (removed row, added "Alias: /bug" to /feedback description) |
| 15 | HIGH | Changed Description | 更新 `/passes` — 从 review passes 重新用于 referral sharing | ✅ COMPLETE (updated description, kept in Model tag) |
| 16 | HIGH | Changed Description | 更新 `/review` — 已弃用，由 `code-review` marketplace plugin 替代 | ✅ COMPLETE (updated description in Project tag) |
| 17 | MED | Changed Description | 更新 `/stickers` — 从 UI sticker packs 更改为订购实体贴纸 | ✅ COMPLETE (updated description in Config tag) |

---

## [2026-03-15 12:50 PM PKT] Claude Code v2.1.76

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Command | 在 Config tag 中添加 `/color [color\|default]` — 为当前会话设置提示栏颜色 | ✅ COMPLETE (added as #4 in Config tag) |
| 2 | HIGH | New Command | 在 Model tag 中添加 `/effort [low\|medium\|high\|max\|auto]` — 设置模型努力级别 | ✅ COMPLETE (added as #38 in Model tag) |
| 3 | MED | Changed Description | 更新 `/status` — 现在是 "打开设置界面（Status 标签页）" 而非 "显示简洁的会话状态摘要" | ✅ COMPLETE (updated description at #20 in Context tag) |
| 4 | MED | Changed Description | 更新 `/desktop` — 现在是 "在 Claude Code Desktop 应用中继续当前会话。仅限 macOS 和 Windows。" | ✅ COMPLETE (updated description at #49 in Remote tag) |
| 5 | LOW | Changed Argument | 更新 `/init` — 官方文档删除了 `[prompt]` 参数提示 | ✅ COMPLETE (removed [prompt] hint at #45 in Project tag) |

---

## [2026-03-17 12:45 PM PKT] Claude Code v2.1.77

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Alias | 为 `/fork` 条目添加 `Alias: /branch`（v2.1.77 将 fork→branch 重命名） | ✅ COMPLETE (added "Alias: /branch" to /fork at #59 in Session tag) |
| 2 | HIGH | New Aliases | 为 8 个 commands 添加别名：`/clear` (+/reset, /new), `/config` (+/settings), `/desktop` (+/app), `/exit` (+/quit), `/rewind` (+/checkpoint), `/resume` (+/continue), `/remote-control` (+/rc), `/mobile` (+/ios, /android) | ✅ COMPLETE (added alias notations to all 8 command descriptions) |
| 3 | MED | Changed Description | 更新 `/diff` — "打开交互式 diff 查看器，显示未提交的更改和每轮 diff" | ✅ COMPLETE (updated description at #44 in Project tag) |
| 4 | MED | Changed Description | 更新 `/memory` — "编辑 CLAUDE.md memory 文件，启用或禁用 auto-memory，查看 auto-memory 条目" | ✅ COMPLETE (updated description at #37 in Memory tag) |
| 5 | MED | Changed Description | 更新 `/copy` — "将最后一个 assistant 响应复制到剪贴板。显示代码块的交互式选择器" | ✅ COMPLETE (updated description at #27 in Export tag) |
| 6 | MED | Changed Description | 更新 `/mobile` — "显示 QR code 以下载 Claude 移动应用" | ✅ COMPLETE (updated description + aliases at #52 in Remote tag) |
| 7 | MED | Changed Description | 更新 `/remote-control` — "使此会话可用于 claude.ai 的远程控制" | ✅ COMPLETE (updated description + alias at #53 in Remote tag) |
| 8 | LOW | Frontmatter Scope | 6 个 skill-only 字段在报告中仍然缺失（有意范围界定） | ❌ INVALID (skill-only fields — same determination as v2.1.74 run) |

---

## [2026-03-18 11:38 PM PKT] Claude Code v2.1.78

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Command | 在 Config tag 中添加 `/voice` — 切换 push-to-talk 语音听写 | ✅ COMPLETE (added as #15 in Config tag) |
| 2 | HIGH | Inverted Alias | 交换 `/fork` → `/branch` 作为主要名称，`/fork` 作为别名 | ✅ COMPLETE (swapped to `/branch` at #56 in Session tag, re-sorted alphabetically) |
| 3 | MED | New Alias | 为 `/permissions` 添加 `/allowed-tools` 别名 | ✅ COMPLETE (added alias to #7 in Config tag) |
| 4 | MED | New Argument | 为 `/copy` 添加 `[N]` 参数语法 | ✅ COMPLETE (updated to `/copy [N]` at #28 in Export tag) |
| 5 | LOW | Frontmatter Scope | 6 个 skill-only 字段在报告中缺失（有意范围界定） | ❌ INVALID (skill-only fields — same determination as v2.1.74 and v2.1.77 runs) |

---

## [2026-03-19 11:54 AM PKT] Claude Code v2.1.79

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | LOW | Frontmatter Scope | 6 个 skill-only 字段在报告中缺失（有意范围界定） | ❌ INVALID (skill-only fields — same determination as v2.1.74, v2.1.77, and v2.1.78 runs) |

---

## [2026-03-20 08:33 AM PKT] Claude Code v2.1.80

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | MED | New Field | 在 frontmatter 表格中添加 `effort` — 当 Command 被调用时覆盖模型努力级别（v2.1.80） | ✅ COMPLETE (added as 5th field, then repositioned to 8th when full field set was added) |
| 2 | HIGH | QA Correction | 添加 6 个缺失字段（`name`, `disable-model-invocation`, `user-invocable`, `context`, `agent`, `hooks`）— 官方文档声明 commands 支持 "与 skills 相同的 frontmatter"；之前的 INVALID 判定（v2.1.74–v2.1.79）不正确 | ✅ COMPLETE (added all 6 fields, count updated 5 → 11, field order matches official docs) |
| 3 | HIGH | Cross-Report Fix | 在 skills 报告（`claude-skills.md`）中添加 `effort` — 该字段在那里也缺失 | ✅ COMPLETE (added as 8th field in skills report, count updated 10 → 11) |

---

## [2026-03-21 09:08 PM PKT] Claude Code v2.1.81

无优先级操作项 — 报告与官方文档完全同步（11 个 frontmatter 字段，63 个内置 commands）。

---

## [2026-03-23 09:48 PM PKT] Claude Code v2.1.81

无优先级操作项 — 报告与官方文档完全同步（11 个 frontmatter 字段，63 个内置 commands）。

---

## [2026-03-25 08:07 PM PKT] Claude Code v2.1.83

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Command | 在 Remote tag 中添加 `/schedule [description]` — 创建、更新、列出或运行 Cloud scheduled tasks | ✅ COMPLETE (added as #56 in Remote tag, count updated 63 → 64) |

---

## [2026-03-26 01:01 PM PKT] Claude Code v2.1.84

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Field | 在 frontmatter 表格中添加 `shell` — `!command` 块的 shell（`bash` 或 `powershell`） | ✅ COMPLETE (added as 12th field before `hooks`, count updated 11 → 12) |
| 2 | LOW | Changed Argument | 为 `/fast` command 添加 `[on\|off]` 参数提示 | ✅ COMPLETE (updated `/fast` to `/fast [on\|off]` at #40 in Model tag) |

---

## [2026-03-27 06:25 PM PKT] Claude Code v2.1.85

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | New Field | 在 frontmatter 表格中添加 `paths` — glob patterns，限制 skill 何时激活 | ✅ COMPLETE (added as 6th field after `user-invocable`, count updated 12 → 13) |

---

## [2026-03-28 06:05 PM PKT] Claude Code v2.1.86

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | MED | Changed Argument | 更新 `/add-dir` — 根据官方文档添加 `<path>` 必需参数提示 | ✅ COMPLETE (updated at #44 in Project tag) |
| 2 | MED | Changed Argument | 更新 `/branch` — 根据官方文档添加 `[name]` 可选参数提示 | ✅ COMPLETE (updated at #57 in Session tag) |
| 3 | MED | Changed Argument | 更新 `/model` — 根据官方文档添加 `[model]` 可选参数提示 | ✅ COMPLETE (updated at #41 in Model tag) |
| 4 | MED | Changed Argument | 更新 `/plan` — 根据官方文档添加 `[description]` 可选参数提示 | ✅ COMPLETE (updated at #43 in Model tag) |
| 5 | MED | Changed Argument | 更新 `/pr-comments` — 根据官方文档添加 `[PR]` 可选参数提示 | ✅ COMPLETE (updated at #47 in Project tag) |
| 6 | MED | Changed Argument | 更新 `/passes` — 移除 `[number]` 参数提示（不在官方文档中） | ✅ COMPLETE (updated at #42 in Model tag) |
| 7 | MED | Changed Argument | 更新 `/rename` — 根据官方文档从 `<name>`（必需）更改为 `[name]`（可选） | ✅ COMPLETE (updated at #62 in Session tag) |
| 8 | LOW | Changed Argument | 更新 `/compact` — 根据官方文档将参数标签从 `[prompt]` 更改为 `[instructions]` | ✅ COMPLETE (updated at #60 in Session tag) |
| 9 | LOW | Changed Argument | 更新 `/feedback` — 根据官方文档将参数标签从 `[description]` 更改为 `[report]` | ✅ COMPLETE (updated at #24 in Debug tag) |

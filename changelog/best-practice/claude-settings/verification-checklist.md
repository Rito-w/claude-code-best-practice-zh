<!-- 翻译自：https://github.com/shanraisshan/claude-code-best-practice/blob/main/changelog/best-practice/claude-settings/verification-checklist.md -->

# Verification Checklist — Settings Report（验证清单 — Settings 报告）

规则随时间累积。每个 workflow-changelog 运行必须执行 ALL 规则在指定深度。当现有规则本应捕获但未捕获（不存在或太浅）的新型漂移时，在此追加新规则。

## 深度级别

| 深度 | 含义 | 示例 |
|-------|---------|---------|
| `exists` | 检查部分/表格/文件是否存在 | "报告有 Sandbox Settings 表格吗？" |
| `presence-check` | 检查特定项目是否存在或不存在 | "`ConfigChange` 事件在 Hook Events 表格中吗？" |
| `content-match` | 逐字与实际值与源进行比较 | "`model` 设置描述与官方文档匹配吗？" |
| `field-level` | 验证每个单独字段都已计入 | "官方文档中的每个 settings key 都出现在报告的正确表格中吗？" |
| `cross-file` | 相同值必须在多个文件中匹配 | "CLAUDE.md hooks 部分与报告的 hook events 匹配吗？" |

---

## 1. Settings Keys Tables（Settings Keys 表格）

验证 settings key 表格与官方文档的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 1A | Key Completeness | 对于官方文档中的每个 settings key，验证它出现在报告的正确部分表格中 | field-level | settings documentation page | 2026-03-05 | 初始清单 — 确保没有新 settings keys 被遗漏 |
| 1B | Key Types | 对于表格中的每个 key，验证 Type 列与官方文档匹配 | content-match | settings documentation page | 2026-03-05 | 初始清单 — 类型不匹配导致用户困惑 |
| 1C | Key Defaults | 对于有默认值的每个 key，验证 Default 列与官方文档匹配 | content-match | settings documentation page | 2026-03-05 | 初始清单 — 错误默认值导致意外行为 |
| 1D | Key Descriptions | 对于每个 key，验证 Description 列准确反映官方文档行为 | content-match | settings documentation page | 2026-03-05 | 初始清单 — 过时描述误导用户 |
| 1E | Scope Column | 对于有 Scope 列的每个 key（MCP, Plugin, Permission 表格），验证 scope 值与官方文档匹配（例如 "Managed only", "Any", "Project"） | content-match | settings documentation page | 2026-03-15 | v2.1.71 发现 `extraKnownMarketplaces` scope 错误（"Any" → "Project"），v2.1.75 发现 `autoMemoryDirectory` scope 限制。没有规则系统验证 scope 列 |
| 1F | Inverse Completeness | 对于报告表格中的每个 key，验证它存在于官方文档中或明确标记为 "not in official docs — unverified"。没有官方支持的项目必须标注 | field-level | settings documentation page + JSON schema | 2026-03-15 | 可疑 keys（`sandbox.ignoreViolations`, `skipWebFetchPreflight` 等）保持 ON HOLD 6 次运行，因为没有规则检查反向 — 报告中不应存在的项目 |
| 1G | Edge-Case Semantics | 对于在边界值（例如 `0`, 空字符串, `null`）有特殊行为的 settings，验证边界行为有文档记录并与官方文档匹配 | content-match | settings documentation page | 2026-03-15 | v2.1.75 发现 `cleanupPeriodDays` 零值行为晚；v2.1.76 添加 "hooks receive empty transcript_path" 详情。边缘情况验证不足 |
| 1H | File Scope Check | 对于报告中列为 `settings.json` key 的每个 key（特别是 Display Settings），验证它确实是 `settings.json` key 而非仅 `~/.claude.json` preference。官方文档将 "Available settings"（settings.json）与 "Global config settings"（~/.claude.json）分开。错误作用域的 keys 误导用户并可能导致 schema validation errors | content-match | settings documentation page "Available settings" vs "Global config settings" sections | 2026-03-18 | v2.1.78 发现 `showTurnDuration` 和 `terminalProgressBarEnabled` 在 Display Settings 中列为 settings.json keys，但官方文档明确说明它们属于 `~/.claude.json` 且 "Adding them to settings.json will trigger a schema validation error." 没有规则验证文件作用域 |

---

## 2. Settings Hierarchy（Settings 层次）

验证 settings 层次表格的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 2A | Priority Levels | 验证层次表格中的所有优先级级别与官方文档匹配（5 级链 + managed policy） | field-level | settings documentation page | 2026-03-05 | 初始清单 — 错误优先级导致覆盖困惑 |
| 2B | File Locations | 对于每个优先级级别，验证文件位置路径与官方文档匹配 | content-match | settings documentation page | 2026-03-05 | 初始清单 — 错误路径导致 settings 被忽略 |
| 2C | Merge Semantics | 验证数组/对象合并行为描述（例如 "concatenated and deduplicated"）与官方文档措辞完全匹配 | content-match | settings documentation page | 2026-03-15 | v2.1.75 发现 "merged" → "concatenated and deduplicated" 更改。没有规则检查合并行为措辞 |
| 2D | Managed Internals | 验证 managed-tier 交付方法（server-managed, MDM, registry, file）和内部优先级顺序与官方文档匹配。验证平台特定文件路径和弃用说明 | field-level | settings documentation page | 2026-03-15 | v2.1.75 用内部优先级和 Windows 路径弃用重构 managed tier。这些子细节没有专用规则 |

---

## 3. Permissions（权限）

验证权限配置准确性的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 3A | Permission Modes | 验证表格中的所有 permission modes 与官方文档匹配 | field-level | settings documentation page | 2026-03-05 | 初始清单 — 缺失 modes 限制用户选项 |
| 3B | Tool Syntax Patterns | 验证所有工具权限语法模式和示例与官方文档匹配 | content-match | settings documentation page | 2026-03-05 | 初始清单 — 错误语法导致权限失败 |
| 3C | Bidirectional Mode Check | 验证报告中的每个 permission mode 存在于官方文档中，且官方文档中的每个 mode 存在于报告中。报告中但不在文档中的 modes 必须标记为 "unverified" | field-level | settings + permissions documentation pages | 2026-03-15 | v2.1.74 发现 `askEdits`/`viewOnly` 在报告中但不在官方文档中 — 它们从运行 1 起未验证。单向检查（docs→report）错过此 3 次运行 |
| 3D | Evaluation Semantics | 验证权限评估顺序（deny-first）、remote-environment 限制和路径前缀解析含义（`//`, `~/`, `/`, `./`）有文档记录并与官方文档匹配 | content-match | permissions documentation page | 2026-03-15 | v2.1.75 发现缺失评估顺序；v2.1.76 发现缺失路径前缀模式。语义行为规则没有专用检查 |

---

## 4. Hooks（重定向）

Hook 分析从此工作流中排除。Hooks 在 [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks) repo 中维护。仅验证重定向链接仍然有效。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 4A | Hooks Redirect | 验证报告中的 hooks 部分包含有效的重定向链接到 claude-code-hooks repo | exists | report file | 2026-03-05 | Hooks 外部化到专用 repo — 仅检查重定向链接有效性 |

---

## 5. Environment Variables（环境变量）

验证环境变量完整性和所有权的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 5A | Env Var Completeness | 验证官方文档中的所有 `env`-configurable environment variables 出现在报告中 | field-level | settings documentation page | 2026-03-05 | 初始清单 — 缺失 env vars 限制用户配置选项 |
| 5B | Ownership Boundary | 验证 `best-practice/claude-cli-startup-flags.md` 中的 env vars 没有复制到 settings 报告中，反之亦然 | cross-file | claude-cli-startup-flags.md vs settings report | 2026-03-05 | 初始清单 — env var 重构将 vars 分到两个文件，必须防止重复 |
| 5C | Env Var Descriptions | 对于表格中的每个 env var，验证描述（格式、值、行为）与官方 /en/env-vars 页面匹配 | content-match | env-vars documentation page | 2026-03-15 | v2.1.74 发现 `ANTHROPIC_CUSTOM_HEADERS` 描述为 "JSON string" 而非 "Name: Value format, newline-separated"。规则 5A 仅检查存在性，不检查描述准确性 |
| 5D | Inverse Env Var Check | 对于报告表格中的每个 env var，验证它存在于官方 /en/env-vars 页面或明确标记为 "not in official docs — unverified" | field-level | env-vars documentation page | 2026-03-15 | v2.1.76 发现 7 个 env vars 在报告中但没有官方支持。没有反向检查，未文档化的 vars 静默累积 |

---

## 6. Examples（示例）

验证示例准确性的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 6A | Quick Reference Example | 验证 Quick Reference 完整示例使用有效当前 settings 并具有正确语法和现实值 | content-match | settings documentation page | 2026-03-05 | 初始清单 — 示例必须展示当前最佳实践 |
| 6B | Example URL Validation | 验证 JSON 示例块中嵌入的任何 URLs（例如 `$schema`, API endpoints）正确解析并使用当前域 | exists | HTTP response | 2026-03-15 | v2.1.74 发现 `$schema` URL 使用错误域（`www.schemastore.org` vs `json.schemastore.org`）。代码块内的 URLs 未被规则 9B 覆盖，该规则仅检查 markdown 链接 |

---

## 7. Cross-File Consistency（跨文件一致性）

验证报告与其他 repo 文件之间一致性的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 7A | CLAUDE.md Sync | 验证 CLAUDE.md 的 Configuration Hierarchy 和 Hooks System 部分与报告一致 | cross-file | CLAUDE.md vs report | 2026-03-05 | 初始清单 — CLAUDE.md 可能与报告漂移 |

---

## 8. Process（流程）

关于工作流验证过程本身的元规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 8A | Source Credibility Guard | 仅当通过官方源（settings documentation page, CLI reference page, GitHub changelog）确认时才将项目标记为漂移。第三方博客源可能过时或错误 — 仅用于线索，在标记前与官方文档验证 | content-match | official docs only | 2026-03-05 | 从 subagents workflow 采用 — 防止博客源的误报 |

---

## 10. Version Metadata & Suspect Key Lifecycle（版本元数据与可疑 Key 生命周期）

验证报告元数据准确性并防止未解决项目无限累积的元规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 10A | Version Metadata | 验证报告的版本徽章、header settings 计数和 env var 计数反映实际审计版本和当前表格行数 | content-match | report file internal consistency | 2026-03-15 | v2.1.71 发现版本徽章不匹配；v2.1.69 发现 header 计数错误。没有规则验证这些元字段 |
| 10B | Suspect Key Escalation | 连续 5 次运行 ON HOLD 后，可疑 keys 必须解决：要么 (a) 通过 JSON schema 确认并标注为 "in JSON schema, not on official page"，要么 (b) 从报告中移除。报告每个可疑 key 的运行计数 | exists | changelog history | 2026-03-15 | 可疑 keys（`sandbox.ignoreViolations`, `skipWebFetchPreflight` 等）跨 6 次运行保持 ON HOLD 且无解决机制。无限累积不提供价值 |
| 10C | Bidirectional Completeness | 通用元规则：报告中的每个 settings key, permission mode, and env var 必须可追溯到官方源或明确标记为 "unverified"。这是规则 1A/3A/5A 的反向。1F, 3C, 5D 的超集 | field-level | official docs vs report | 2026-03-15 | 从研究中合成的交叉规则：6 个项目被发现晚因为只有 docs→report 检查存在。反向（report→docs）捕获孤立条目 |

---

## 9. Hyperlinks（超链接）

验证报告中所有超链接有效性的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 9A | Local File Links | 验证所有相对文件链接解析到存在的文件 | exists | local filesystem | 2026-03-05 | 初始清单 — 文件移动可能破坏相对链接 |
| 9B | External URL Links | 验证所有外部 URLs 返回有效页面（非 404 或错误） | exists | HTTP response | 2026-03-05 | 初始清单 — 外部文档页面可能被重组或移除 |
| 9C | Anchor Links | 验证所有内部 anchor 链接指向同一文件中存在的标题 | exists | file headings | 2026-03-05 | 初始清单 — 部分重命名可能破坏 anchor 链接 |

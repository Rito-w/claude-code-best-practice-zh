<!-- 翻译自：https://github.com/shanraisshan/claude-code-best-practice/blob/main/changelog/best-practice/claude-subagents/verification-checklist.md -->

# Verification Checklist — Subagents Report（验证清单 — Subagents 报告）

规则随时间累积。每个 workflow-changelog 运行必须执行 ALL 规则在指定深度。当现有规则本应捕获但未捕获（不存在或太浅）的新型漂移时，在此追加新规则。

## 深度级别

| 深度 | 含义 | 示例 |
|-------|---------|---------|
| `exists` | 检查部分/表格/文件是否存在 | "报告有 Memory Scopes 表格吗？" |
| `presence-check` | 检查特定项目是否存在或不存在 | "`color` 字段在 Frontmatter Fields 表格中吗？" |
| `content-match` | 逐字与实际值与源进行比较 | "`model` 字段描述与官方文档匹配吗？" |
| `field-level` | 验证每个单独字段都已计入 | "官方文档中的每个 frontmatter 字段都出现在表格中吗？" |
| `cross-file` | 相同值必须在多个文件中匹配 | "CLAUDE.md agent 部分与报告的字段列表匹配吗？" |

---

## 1. Frontmatter Fields Table（Frontmatter 字段表格）

验证 Frontmatter Fields 表格与官方文档的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 1A | Field Completeness | 对于官方文档中的每个 agent frontmatter 字段，验证它出现在报告的 Frontmatter Fields 表格中 | field-level | sub-agents reference page | 2026-02-28 | 初始清单 — 确保没有新字段被遗漏 |
| 1B | Field Types | 对于表格中的每个字段，验证 Type 列与官方文档匹配 | content-match | sub-agents reference page | 2026-02-28 | 初始清单 — 类型不匹配导致用户困惑 |
| 1C | Required Status | 对于每个字段，验证 Required 列与官方文档匹配 | content-match | sub-agents reference page | 2026-02-28 | 初始清单 — 错误的 required 状态导致 agent 损坏 |
| 1D | Field Descriptions | 对于每个字段，验证 Description 列准确反映官方文档行为 | content-match | sub-agents reference page | 2026-02-28 | 初始清单 — 过时描述误导用户 |

---

## 2. Memory Scopes（Memory 作用域）

验证 Memory Scopes 表格的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 2A | Scope Completeness | 验证官方文档中的所有 memory scopes 出现在 Memory Scopes 表格中 | field-level | sub-agents reference page | 2026-02-28 | 初始清单 — 可能添加新 scopes |
| 2B | Storage Locations | 对于每个 scope，验证 Storage Location 列与官方文档匹配 | content-match | sub-agents reference page | 2026-02-28 | 初始清单 — 错误路径导致数据丢失 |

---

## 3. Examples（示例）

验证示例准确性的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 3A | Minimal Example | 验证 minimal example 仅使用 required fields 并具有有效语法 | content-match | sub-agents reference page | 2026-02-28 | 初始清单 — minimal example 应保持 minimal |
| 3B | Full-Featured Example | 验证 full-featured example 展示 ALL available frontmatter fields | field-level | sub-agents reference page | 2026-02-28 | 初始清单 — 完整示例必须展示每个字段 |

---

## 4. Scope & Priority（作用域与优先级）

验证作用域和优先级信息的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 4A | Priority Order | 验证 Scope and Priority 表格按正确优先级顺序列出所有 agent 位置 | content-match | sub-agents reference page + CLI reference page | 2026-02-28 | 初始清单 — 错误优先级顺序导致解析错误 |
| 4B | Invocation Methods | 验证 invocation methods 表格列出 CLI reference 和 sub-agents docs 中的所有 invocation 方法，包括 `--agent`（单数）、`--agents`（复数）、`/agents`、`claude agents`、Agent tool 和 agent resumption | field-level | CLI reference page + sub-agents reference page | 2026-03-07 | `--agent` CLI flag 在 invocation 表格中缺失 — 它是作为特定 agent 运行 Claude 的独特 invocation 方法 |

---

## 5. Cross-File Consistency（跨文件一致性）

验证报告与其他 repo 文件之间一致性的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 5A | CLAUDE.md Sync | 验证 CLAUDE.md 的 Subagent Definition Structure 部分列出与报告 Frontmatter Fields 表格相同的字段 | cross-file | CLAUDE.md vs report | 2026-02-28 | 初始清单 — CLAUDE.md 可能与报告漂移 |

---

## 6. Process（流程）

关于工作流验证过程本身的元规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 6A | Source Credibility Guard | 仅当通过官方源（sub-agents reference page, CLI reference page, GitHub changelog）确认时才将项目标记为漂移。第三方博客源可能过时或错误 — 仅用于线索，在标记前与官方文档验证 | content-match | official docs only | 2026-02-28 | 从 hooks workflow 采用 — 防止博客源的误报 |

---

## 7. Agent Tables（Agent 表格）

验证 Official Claude Agents 和 Agents in This Repository 表格的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 7A | Built-in Agent Completeness | 验证 "Official Claude Agents" 表格列出所有内置 agent 类型，具有正确的 model, tools 和 description | field-level | sub-agents reference page + changelog | 2026-02-28 | 报告只有 3/5 个内置 agents — `claude-code-guide` 和 `statusline-setup` 缺失 |
| 7B | Repository Agent Completeness | 扫描 `.claude/agents/**/*.md` 并验证每个 agent 文件出现在 "Agents in This Repository" 表格中，具有正确的 model, color, tools, skills 和 memory 列 | field-level | `.claude/agents/**/*.md` file frontmatter | 2026-02-28 | Repo agents 是手动维护的 — 添加到 repo 的新 agents 未反映在报告中 |
| 7C | Repository Agent Links | 验证 "Agents in This Repository" 表格中的每个 agent 名称有可点击链接解析到正确的 `.md` 文件 | exists | resolved file path from `best-practice/` | 2026-02-28 | Agent 名称可点击 — 文件移动后链接必须保持有效 |

---

## 8. Hyperlinks（超链接）

验证报告中所有超链接有效性的规则。

| # | 类别 | 检查 | 深度 | 对比源 | 添加日期 | 来源 |
|---|----------|-------|-------|-----------------|-------|--------|
| 8A | Local File Links | 验证所有相对文件链接（例如 `../.claude/agents/weather-agent.md`）解析到存在的文件 | exists | local filesystem | 2026-02-28 | 文件移动（reports/ → best-practice/）破坏了相对链接 — 必须捕获未来损坏 |
| 8B | External URL Links | 验证所有外部 URLs（例如 `https://code.claude.com/docs/en/sub-agents`）返回有效页面 | exists | HTTP response | 2026-02-28 | 外部文档页面可能被重组或移除 — 必须在每次运行中验证 |
| 8C | Cross-File Reference Links | 验证到其他报告文件的链接（例如 `../reports/claude-agent-memory.md`）解析到存在的文件 | exists | local filesystem | 2026-02-28 | 报告可能被移动或重命名 — 交叉引用必须保持同步 |

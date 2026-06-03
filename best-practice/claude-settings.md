# Settings 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Jun%2002%2C%202026%2010%3A48%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.160-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../.claude/settings.json)

Claude Code `settings.json` 文件中所有可用配置选项的综合指南。截至 v2.1.160，Claude Code 提供 **80+ 设置项**和 **200+ 环境变量**（使用 `settings.json` 中的 `"env"` 字段可避免使用包装脚本）。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Table of Contents

1. [Settings Hierarchy](#settings-hierarchy)
2. [Core Configuration](#core-configuration)
3. [Permissions](#permissions)
4. [Hooks](#hooks)
5. [MCP Servers](#mcp-servers)
6. [Sandbox](#sandbox)
7. [Plugins](#plugins)
8. [Model Configuration](#model-configuration)
9. [Display & UX](#display--ux)
10. [AWS & Cloud Credentials](#aws--cloud-credentials)
11. [Environment Variables](#environment-variables-via-env)
12. [Useful Commands](#useful-commands)

---

## Settings Hierarchy

Settings apply in order of precedence (highest to lowest):

| Priority | Location | Scope | Shared? | Purpose |
|----------|----------|-------|---------|---------|
| 1 | Managed settings | Organization | Yes (deployed by IT) | Security policies that cannot be overridden |
| 2 | Command line arguments | Session | N/A | Temporary single-session overrides |
| 3 | `.claude/settings.local.json` | Project | No (git-ignored) | Personal project-specific |
| 4 | `.claude/settings.json` | Project | Yes (committed) | Team-shared settings |
| 5 | `~/.claude/settings.json` | User | N/A | Global personal defaults |

**Managed settings** are organization-enforced and cannot be overridden by any other level, including command line arguments. Delivery methods:
- **Server-managed** settings (remote delivery)
- **MDM profiles** — macOS plist at `com.anthropic.claudecode`
- **Registry policies** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode` (admin) and `HKCU\SOFTWARE\Policies\ClaudeCode` (user-level, lowest policy priority)
- **File** — `managed-settings.json` and `managed-mcp.json` (macOS: `/Library/Application Support/ClaudeCode/`, Linux/WSL: `/etc/claude-code/`, Windows: `C:\Program Files\ClaudeCode\`)
- **Drop-in directory** — `managed-settings.d/` alongside `managed-settings.json` for independent policy fragments (v2.1.83). Following the systemd convention, `managed-settings.json` is merged first as the base, then all `*.json` files in the drop-in directory are sorted alphabetically and merged on top. Later files override earlier ones for scalar values; arrays are concatenated and de-duplicated; objects are deep-merged. Hidden files starting with `.` are ignored. Use numeric prefixes to control merge order (e.g., `10-telemetry.json`, `20-security.json`)

Within the managed tier, precedence is: server-managed > MDM/OS-level policies > file-based (`managed-settings.d/*.json` + `managed-settings.json`) > HKCU registry (Windows only). Only one managed source is used; sources do not merge across tiers. Within the file-based tier, drop-in files and the base file are merged together.

> **Note:** As of v2.1.75, the deprecated Windows fallback path `C:\ProgramData\ClaudeCode\managed-settings.json` has been removed. Use `C:\Program Files\ClaudeCode\managed-settings.json` instead.

> **Note (v2.1.126):** `/config` now persists changes to `~/.claude/settings.json` instead of holding them in memory only. Edits made through the interactive Config UI survive restarts.

### 动态与父级策略（仅限 Managed）

这些键位于 managed 层级，用于控制 managed 设置本身的计算和合并方式。它们不会出现在用户、项目或本地设置中。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `parentSettingsBehavior` | string | `"first-wins"` | 控制 SDK `managedSettings` 父级如何与本地 managed 文件合并。`"first-wins"` 保持现有优先级——第一个非空的 managed 源提供所有值。`"merge"` 将父级深度合并到本地 managed 文件之上，使管理员可以在 managed 基础之上叠加组织范围的策略 (v2.1.133) |
| `policyHelper` | object | - | 在运行时动态计算 managed 设置的 managed 可执行程序。对象字段：`path`（字符串——helper 二进制的绝对路径）、`timeoutMs`（数字——超过此毫秒后中止 helper）、`refreshIntervalMs`（数字——超过此毫秒后重新运行 helper 以刷新策略）。输出解析为 JSON，视为 `managed-settings.json` 的内容。用于从外部系统（LDAP 组、资产数据库等）计算组织策略，无需重新部署静态文件 (v2.1.136) |

**Important**:
- `deny` rules have highest safety precedence and cannot be overridden by lower-priority allow/ask rules.
- Managed settings may lock or override local behavior even if local files specify different values.
- Array settings (e.g., `permissions.allow`) are **concatenated and deduplicated** across scopes — entries from all levels are combined, not replaced.

---

## Core Configuration

### General Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `$schema` | string | - | JSON Schema URL for IDE validation and autocompletion (e.g., `"https://json.schemastore.org/claude-code-settings.json"`) |
| `model` | string | `"default"` | Override default model. Accepts aliases (`sonnet`, `opus`, `haiku`) or full model IDs |
| `agent` | string | - | Set the default agent for the main conversation. Value is the agent name from `.claude/agents/`. Also available via `--agent` CLI flag |
| `language` | string | `"english"` | Claude's preferred response language. Also sets the voice dictation language and the terminal tab title (v2.1.121) |
| `claudeMdExcludes` | array | - | 加载 [memory](https://code.claude.com/docs/en/memory) 时跳过的 `CLAUDE.md` 文件的 glob 模式或绝对路径。模式匹配绝对文件路径。仅适用于用户、项目和本地 memory；无法排除托管策略文件。示例：`["**/vendor/**/CLAUDE.md"]` |
| `claudeMd` | string | - | **（仅托管）** 以组织托管 [memory](https://code.claude.com/docs/en/memory) 形式注入的 CLAUDE.md 风格指令。仅在托管或策略设置中生效；在用户、项目和本地设置中被忽略。示例：`"Always run make lint before committing."` |
| `cleanupPeriodDays` | number | `30` | Age cutoff for the startup cleanup sweep (minimum 1). Inactive session transcripts and orphaned subagent worktrees are deleted; as of v2.1.117 the sweep also covers `~/.claude/tasks/`, `~/.claude/shell-snapshots/`, and `~/.claude/backups/`. Setting to `0` is rejected with a validation error. To disable transcript writes in non-interactive mode (`-p`), use `--no-session-persistence` or `persistSession: false` SDK option |
| `autoUpdatesChannel` | string | `"latest"` | Release channel: `"stable"` or `"latest"` |
| `minimumVersion` | string | - | Prevent the auto-updater from downgrading below a specific version. Automatically set when switching to the stable channel and choosing to stay on the current version until stable catches up. Used with `autoUpdatesChannel` |
| `alwaysThinkingEnabled` | boolean | `false` | Enable extended thinking by default for all sessions |
| `skipWebFetchPreflight` | boolean | `false` | Skip WebFetch blocklist check before fetching URLs *(in JSON schema, not on official settings page)* |
| `availableModels` | array | - | Restrict which models users can select via `/model`, `--model`, Config tool, or `ANTHROPIC_MODEL`. Does not affect the Default option. Example: `["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | Require users to opt in to fast mode each session |
| `defaultShell` | string | `"bash"` | Default shell for input-box `!` commands. Accepts `"bash"` (default) or `"powershell"`. Setting `"powershell"` routes interactive `!` commands through PowerShell on Windows. Requires `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` (v2.1.84). **v2.1.120:** When PowerShell is available, it is used as the fallback shell on Windows even without Git for Windows installed. **v2.1.126:** When PowerShell is enabled, it is treated as the *primary* shell instead of defaulting to Bash. PowerShell 7 detection now also covers Microsoft Store installs, MSI installs not on PATH, and `.NET` global tool installs |
| `includeGitInstructions` | boolean | `true` | Include built-in commit and PR workflow instructions and the git status snapshot in Claude's system prompt. The `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` environment variable takes precedence over this setting when set |
| `voice` | object | - | Voice dictation configuration. Object with three fields: `enabled` (boolean — push-to-talk on/off), `mode` (string — `"hold"` for hold-to-talk or `"tap"` for tap-to-toggle), and `autoSubmit` (boolean — submit transcript immediately when dictation ends). Written automatically when you run `/voice`. Requires a Claude.ai account (v2.1.118 expanded structure) |
| `voiceEnabled` | boolean | - | **DEPRECATED** — legacy alias for `voice.enabled`. Use the `voice` object instead to get `mode` and `autoSubmit` controls |
| `showClearContextOnPlanAccept` | boolean | `false` | Show the "clear context" option on the plan accept screen. Set to `true` to restore the option (hidden by default since v2.1.81) |
| `viewMode` | string | - | Default transcript view mode on startup: `"default"`, `"verbose"`, or `"focus"`. Overrides the sticky Ctrl+O selection when set |
| `disableDeepLinkRegistration` | string | - | Set to `"disable"` to prevent Claude Code from registering the `claude-cli://` protocol handler with the operating system on startup. Deep links let external tools open a Claude Code session with a pre-filled prompt via `claude-cli://open?q=...`. The `q` parameter supports multi-line prompts using URL-encoded newlines (`%0A`). Useful in environments where protocol handler registration is restricted or managed separately |
| `showThinkingSummaries` | boolean | `false` | Show extended thinking summaries in interactive sessions. When unset or `false` (default in interactive mode), thinking blocks are redacted by the API and shown as a collapsed stub. Redaction only changes what you see, not what the model generates — to reduce thinking spend, lower the budget or disable thinking instead. Non-interactive mode (`-p`) and SDK callers always receive summaries regardless of this setting |
| `disableSkillShellExecution` | boolean | `false` | Disable inline shell execution for `` !`...` `` and `` ```! `` blocks in skills and custom commands from user, project, plugin, or additional-directory sources. Commands are replaced with `[shell command execution disabled by policy]` instead of being run. Bundled and managed skills are not affected (v2.1.91) |
| `skillOverrides` | string | - | 控制自动技能调用行为。值：`"off"`（完全不调用技能）、`"user-invocable-only"`（仅运行用户通过 `/skill-name` 明确调用的技能；禁用通过技能描述的自动发现）、`"name-only"`（仅按精确名称匹配技能；禁用基于描述的自动发现）。用于更严格地控制模型加载或运行哪些技能 (v2.1.129) |
| `maxSkillDescriptionChars` | number | `1536` | 每个技能在 [技能列表](https://code.claude.com/docs/en/skills) 中 `description` 和 `when_to_use` 文本合并后的字符上限。超出此长度的文本将被截断 (v2.1.105) |
| `skillListingBudgetFraction` | number | `0.01` | 预留用于每轮可见 [技能列表](https://code.claude.com/docs/en/skills) 的模型上下文窗口比例（`0.01` = 1%）。当列表超出预算时，最少使用的技能描述将折叠为纯名称，Claude 仍可调用但看不到说明 (v2.1.105) |
| `forceRemoteSettingsRefresh` | boolean | `false` | **(Managed only)** Block CLI startup until remote managed settings are freshly fetched. If the fetch fails, the CLI exits (fail-closed). Use in enterprise environments where policy enforcement must be up-to-date before any session begins (v2.1.92) |
| `wslInheritsWindowsSettings` | boolean | `false` | **(Windows managed settings only)** When `true`, Claude Code on WSL reads managed settings from the Windows policy chain (HKLM registry + `C:\Program Files\ClaudeCode\managed-settings.json`) in addition to `/etc/claude-code`, with Windows sources taking priority. Only honored when set in the HKLM registry key or `C:\Program Files\ClaudeCode\managed-settings.json`, both of which require Windows admin to write. For HKCU policy to also apply on WSL, the flag must additionally be set in HKCU itself. Has no effect on native Windows (v2.1.118) |
| `tui` | string | `"default"` | Rendering mode: `"fullscreen"` or `"default"`. Set via `/tui fullscreen` for flicker-free alt-screen rendering (v2.1.110) |
| `awaySummaryEnabled` | boolean | `true` | Generate an "away summary" (idle-session recap) when the user returns after being away. Set to `false` to opt out. Pairs with the `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` env var (v2.1.110) |
| `feedbackSurveyRate` | number | - | Probability (0–1) that the session quality survey appears when eligible. Enterprise admins can control how often the survey is shown. Example: `0.05` = 5% of eligible sessions |

**Example:**
```json
{
  "model": "opus",
  "agent": "code-reviewer",
  "language": "japanese",
  "cleanupPeriodDays": 60,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true
}
```

### Plans & Memory Directories

Store plan and auto-memory files in custom locations.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | Directory where `/plan` outputs are stored |
| `autoMemoryDirectory` | string | - | Custom directory for auto-memory storage. Accepts `~/`-expanded paths. Not accepted in project settings (`.claude/settings.json`) to prevent redirecting memory writes to sensitive locations; accepted from policy, local, and user settings |
| `autoMemoryEnabled` | boolean | `true` | 启用 [auto memory](https://code.claude.com/docs/en/memory)。设为 `false` 时，Claude 不会读取或写入 auto-memory 目录。也可在会话中通过 `/memory` 切换，或通过 `CLAUDE_CODE_DISABLE_AUTO_MEMORY` 环境变量禁用 |

**Example:**
```json
{
  "plansDirectory": "./my-plans"
}
```

**Use Case:** Useful for organizing planning artifacts separately from Claude's internal files, or for keeping plans in a shared team location.

### Worktree Settings

Configure how `--worktree` creates and manages git worktrees. Useful for reducing disk usage and startup time in large monorepos.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | Directories to symlink from the main repository into each worktree to avoid duplicating large directories on disk |
| `worktree.sparsePaths` | array | `[]` | Directories to check out in each worktree via git sparse-checkout (cone mode). Only the listed paths are written to disk |
| `worktree.baseRef` | string | `"fresh"` | 新 worktree 的分支来源：`"fresh"` 从主分支 HEAD 的最新 fetch 创建 worktree；`"head"` 从调用仓库的当前 HEAD 分支。当你希望 worktree 继承你进行中的工作时使用 `"head"` (v2.1.133) |
| `worktree.bgIsolation` | string | `"worktree"` | [后台会话](https://code.claude.com/docs/en/agent-view)的隔离模式。`"worktree"`（默认）在主 checkout 中调用 `EnterWorktree` 前阻止 `Edit`/`Write`；`"none"` 允许后台任务直接编辑工作副本 (v2.1.143) |

**Example:**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### Attribution Settings

Customize attribution messages for git commits and pull requests.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `attribution.commit` | string | Co-authored-by | Git commit attribution (supports trailers) |
| `attribution.pr` | string | Generated message | Pull request description attribution |
| `prUrlTemplate` | string | - | URL template that controls how the "PR" badge in commit attribution links to the pull request UI. Supports placeholders for the repo host, owner, repo, and PR number. Useful for self-hosted GitLab/Bitbucket/GitHub Enterprise instances where the default `https://github.com/...` URL does not apply (v2.1.119) |
| `includeCoAuthoredBy` | boolean | `true` | **DEPRECATED** - Use `attribution` instead |

**Example:**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**Note:** Set to empty string (`""`) to hide attribution entirely.

### Authentication Helpers

Scripts for dynamic authentication token generation.

| Key | Type | Description |
|-----|------|-------------|
| `apiKeyHelper` | string | Shell script path that outputs auth token (sent as `X-Api-Key` header) |
| `forceLoginMethod` | string | Restrict login to `"claudeai"` or `"console"` accounts |
| `forceLoginOrgUUID` | string \| array | Require login to belong to a specific organization. Accepts a single UUID string (which also pre-selects that organization during login) or an array of UUIDs where any listed organization is accepted without pre-selection. When set in managed settings, login fails if the authenticated account does not belong to a listed organization; an empty array fails closed and blocks login with a misconfiguration message |

**Example:**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### Company Announcements

Display custom announcements to users at startup (cycled randomly).

| Key | Type | Description |
|-----|------|-------------|
| `companyAnnouncements` | array | Array of strings displayed at startup |

**Example:**
```json
{
  "companyAnnouncements": [
    "Welcome to Acme Corp!",
    "Remember to run tests before committing!",
    "Check the wiki for coding standards"
  ]
}
```

---

## Permissions

Control what tools and operations Claude can perform.

### Permission Structure

```json
{
  "permissions": {
    "allow": [],
    "ask": [],
    "deny": [],
    "additionalDirectories": [],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### Permission Keys

| Key | Type | Description |
|-----|------|-------------|
| `permissions.allow` | array | Rules allowing tool use without prompting |
| `permissions.ask` | array | Rules requiring user confirmation |
| `permissions.deny` | array | Rules blocking tool use (highest precedence) |
| `permissions.additionalDirectories` | array | Extra directories Claude can access |
| `permissions.defaultMode` | string | Default permission mode. In Remote environments, only `acceptEdits` and `plan` are honored (v2.1.70+) |
| `permissions.disableBypassPermissionsMode` | string | Prevent bypass mode activation |
| `permissions.skipDangerousModePermissionPrompt` | boolean | Skip the confirmation prompt shown before entering bypass permissions mode via `--dangerously-skip-permissions` or `defaultMode: "bypassPermissions"`. Ignored when set in project settings (`.claude/settings.json`) to prevent untrusted repositories from auto-bypassing the prompt |
| `allowManagedPermissionRulesOnly` | boolean | **(Managed only)** Only managed permission rules apply; user/project `allow`, `ask`, `deny` rules are ignored |
| `autoMode` | object | 自定义 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 分类器阻止和允许的操作。包含 `environment`（受信任的基础设施描述）、`allow`（阻止规则的例外）、`soft_deny`（阻止规则）和 `hard_deny`（无条件阻止规则——与 `soft_deny` 同级但不能被 `allow` 例外或 `"$defaults"` 哨兵覆盖；v2.1.136）——均为散文字符串数组。**不从共享项目设置**（`.claude/settings.json`）中读取以防止仓库注入。可在用户、本地和 managed 设置中使用。设置 `allow`、`soft_deny` 或 `hard_deny` 会**替换**该部分的整个默认列表，除非你在数组中包含字面字符串 `"$defaults"`——该哨兵在该位置继承内置规则，使自定义条目与其并列 (v2.1.118)。在自定义之前运行 `claude auto-mode defaults` 查看内置规则 |
| `disableAutoMode` | string | Set to `"disable"` to prevent [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) from being activated. Removes `auto` from the `Shift+Tab` cycle and rejects `--permission-mode auto` at startup. Can be set at any settings level; most useful in managed settings where users cannot override it |
| `useAutoModeDuringPlan` | boolean | Whether plan mode uses auto mode semantics when auto mode is available. Default: `true`. Not read from shared project settings (`.claude/settings.json`). Appears in `/config` as "Use auto mode during plan" |

### Permission Modes

| Mode | Behavior |
|------|----------|
| `"default"` | Standard permission checking with prompts |
| `"acceptEdits"` | 自动接受文件编辑 **和工作目录或 `additionalDirectories` 中路径的常见文件系统命令**（`mkdir`、`touch`、`mv`、`cp` 等）。**v2.1.160：** 写入构建工具配置文件（`.npmrc`、`.yarnrc*`、`bunfig.toml`、`.bazelrc`、`.pre-commit-config.yaml`、`.devcontainer/` 等，这些文件会授予代码执行权限）和 shell 启动文件（`.zshenv`、`.zlogin`、`.bash_login`）以及 `~/.config/git/` 之前始终会提示 |
| `"dontAsk"` | Auto-denies tools unless pre-approved via `/permissions` or `permissions.allow` rules |
| `"bypassPermissions"` | 跳过所有权限检查（危险）。写入受保护路径（`.git`、`.claude`、`.vscode`、`.idea`、`.husky`）仍会提示。截至 v2.1.121，写入 `.claude/commands/`、`.claude/agents/`、`.claude/skills/` 和 `.claude/worktrees/` 明确豁免受保护路径提示，因为 Claude 在创建 skills、子代理和 commands 时会常规写入这些地方。**v2.1.126** 进一步扩展豁免：在 `--dangerously-skip-permissions` 下写入 `.claude/`、`.git/`、`.vscode/` 和 shell 配置文件（如 `.bashrc`、`.zshrc`）不再提示。针对文件系统根目录或 home 目录的删除命令（`rm -rf /`、`rm -rf ~`）仍会提示，作为防止模型错误的断路器 |
| `"auto"` | Auto-approves tool calls with background safety checks that verify actions align with your request. Research preview. Classifier auto-approves read-only and file edits; sends everything else through a safety check. Falls back to prompting after 3 consecutive or 20 total blocks. In the default `Shift+Tab` permission-mode cycle since v2.1.111 (the `--enable-auto-mode` flag was removed in v2.1.111 — start in this mode with `--permission-mode auto`). Configure with the `autoMode` setting |
| `"plan"` | Read-only exploration mode |

### Tool Permission Syntax

| Tool | Syntax | Examples |
|------|--------|----------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`, `Bash(* install)`, `Bash(git * main)` |
| `PowerShell` | `PowerShell(cmd *)` | `PowerShell(Get-ChildItem *)`, `PowerShell(git commit *)` — 与 Bash 形状相同；常见别名会被规范化（`gci`/`ls`/`dir` → `Get-ChildItem`），且会解析 PowerShell AST，因此 `|`/`;`/`&&`/`||` 链的每个子命令都必须匹配 |
| `Read` | `Read(path pattern)` | `Read(.env)`, `Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`, `Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`, `Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | Global web search |
| `Task` | `Task(agent-name)` | `Task(Explore)`, `Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`, `Agent(*)` — permission scoped to subagent spawning |
| `Skill` | `Skill(skill-name)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` or `MCP(server:tool)` | `mcp__memory__*`, `MCP(github:*)` |

**Evaluation order:** Rules are evaluated in order: deny rules first, then ask, then allow. The first matching rule wins.

**Read/Edit path patterns:** Permission rules for `Read`, `Edit`, and `Write` support gitignore-style patterns with four prefix types:

| Prefix | Meaning | Example |
|--------|---------|---------|
| `//` | Absolute path from filesystem root | `Read(//Users/alice/file)` |
| `~/` | Relative to home directory | `Read(~/.zshrc)` |
| `/` | Relative to project root | `Edit(/src/**)` |
| `./` or none | Relative path (current directory) | `Read(.env)`, `Read(*.ts)` |

**Symlink 解析：** 权限规则同时检查 symlink 路径和其解析目标。**允许**规则仅在 symlink 和目标 *都* 匹配时生效 — 允许目录内的 symlink 指向外部仍会提示。**拒绝**规则在 symlink 或目标 *任一* 匹配时生效 — 指向被拒绝文件的 symlink 本身也被拒绝。

**Bash wildcard notes:**
- `*` can appear at **any position**: prefix (`Bash(* install)`), suffix (`Bash(npm *)`), or middle (`Bash(git * main)`)
- **Word boundary:** `Bash(ls *)` (space before `*`) matches `ls -la` but NOT `lsof`; `Bash(ls*)` (no space) matches both
- `Bash(*)` is treated as equivalent to `Bash` (matches all bash commands)
- Permission rules support output redirections: `Bash(python:*)` matches `python script.py > output.txt`
- The legacy `:*` suffix syntax (e.g., `Bash(npm:*)`) is equivalent to ` *` but is deprecated
- **复合命令：** shell 运算符（`&&`、`||`、`;`、`|`、`|&`、`&` 和换行）将命令拆分，每个子命令必须独立匹配 — `Bash(safe-cmd *)` **不**授权 `safe-cmd && other-cmd`
- **进程包装器：** `timeout`、`time`、`nice`、`nohup` 和 `stdbuf` 在匹配前会被剥离（因此 `Bash(npm test *)` 也匹配 `timeout 30 npm test`）； bare `xargs`（无标志）也会被剥离。执行包装器 `watch`、`setsid`、`ionice`、`flock` 和带 `-exec`/`-delete` 的 `find` 始终会提示，无法通过前缀规则批准

**Example:**
```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(git push *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)",
      "Bash(curl *)"
    ],
    "additionalDirectories": ["../shared-libs/"]
  }
}
```

---

## Hooks

Hook configuration (events, properties, matchers, exit codes, environment variables, and HTTP hooks) is maintained in a dedicated repository:

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — Complete hook reference with sound notification system, all 25 hook events, HTTP hooks, matcher patterns, exit codes, and environment variables.

Hook-related settings keys (`hooks`, `disableAllHooks` (also disables any custom status line), `allowManagedHooksOnly`, `allowedHttpHookUrls`, `httpHookAllowedEnvVars`) are documented there.

For the official hooks reference, see the [Claude Code Hooks Documentation](https://code.claude.com/docs/en/hooks).

---

## MCP Servers

Configure Model Context Protocol servers for extended capabilities.

> **OAuth (v2.1.111):** MCP servers that authenticate via OAuth follow [RFC 9728](https://datatracker.ietf.org/doc/rfc9728/) for protected-resource metadata discovery. Compliant servers expose authorization endpoints under `/.well-known/oauth-protected-resource`, and Claude Code completes the OAuth flow automatically — no manual `apiKeyHelper` or `headersHelper` script required for spec-conformant servers.

### MCP Settings

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enableAllProjectMcpServers` | boolean | Any | Auto-approve all `.mcp.json` servers |
| `enabledMcpjsonServers` | array | Any | Allowlist specific server names |
| `disabledMcpjsonServers` | array | Any | Blocklist specific server names |
| `allowedMcpServers` | array | Managed only | Allowlist with name/command/URL matching |
| `deniedMcpServers` | array | Managed only | Blocklist with matching |
| `allowManagedMcpServersOnly` | boolean | Managed only | Only allow MCP servers explicitly listed in managed allowlist |
| `allowAllClaudeAiMcps` | boolean | 仅托管 | 将 claude.ai 云端 MCP 连接器与 `managed-mcp.json` 一起加载。启用后，claude.ai 托管的 MCP 连接器将与管理员部署的托管 MCP 服务器一同可用 *（v2.1.150 更新日志中出现，尚未在官方设置页面）* |
| `channelsEnabled` | boolean | Managed only | Allow [channels](https://code.claude.com/docs/en/channels) for Team and Enterprise users. When unset or `false`, channel message delivery is blocked regardless of `--channels` flag |
| `allowedChannelPlugins` | array | Managed only | Allowlist of channel plugins that may push messages. Replaces the default Anthropic allowlist when set. Undefined = fall back to the default, empty array = block all channel plugins. Requires `channelsEnabled: true`. Each entry is an object with `marketplace` and `plugin` fields (v2.1.84) |

### MCP Server Matching (Managed Settings)

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": "npx @modelcontextprotocol/*" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" }
  ]
}
```

### Per-Server Tool Loading (`alwaysLoad`, v2.1.121)

By default, MCP tool definitions are deferred (loaded into context on demand via tool search). Set `alwaysLoad: true` on an individual MCP server entry in `.mcp.json` (or inline `mcpServers`) to exempt that server from deferral — every tool from that server then loads upfront at session start regardless of `ENABLE_TOOL_SEARCH`. Available on all server types; requires Claude Code v2.1.121+. Use this only for a small set of tools needed every turn — each upfront tool consumes context that would otherwise be available for conversation.

```json
{
  "mcpServers": {
    "always-on-server": {
      "type": "http",
      "url": "https://mcp.example.com",
      "alwaysLoad": true
    }
  }
}
```

An MCP server can also mark individual tools as always-loaded by including `"anthropic/alwaysLoad": true` in the tool's `_meta` object — useful when only a subset of a server's tools should bypass deferral.

**Example:**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## Sandbox

Configure bash command sandboxing for security.

### Sandbox Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `sandbox.enabled` | boolean | `false` | Enable bash sandboxing |
| `sandbox.failIfUnavailable` | boolean | `false` | Exit with error when sandbox is enabled but cannot start, instead of running unsandboxed. Useful for enterprise policies that require strict sandboxing (v2.1.83) |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | Auto-approve bash when sandboxed |
| `sandbox.excludedCommands` | array | `[]` | Commands to run outside sandbox |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | Allow `dangerouslyDisableSandbox`. When set to `false`, the escape hatch is completely disabled and all commands must run sandboxed (or be in `excludedCommands`). Useful for enterprise policies that require strict sandboxing |
| `sandbox.ignoreViolations` | object | `{}` | Map of command patterns to path arrays — suppress violation warnings *(in JSON schema, not on official settings page)* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | **(Linux and WSL2 only)** Enable weaker sandbox for unprivileged Docker environments (reduces security) |
| `sandbox.network.allowUnixSockets` | array | `[]` | **(macOS only)** Specific Unix socket paths accessible in sandbox. Ignored on Linux and WSL2, where the seccomp filter cannot inspect socket paths; use `allowAllUnixSockets` instead |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | Allow all Unix sockets (overrides `allowUnixSockets`). On Linux and WSL2 this is the only way to permit Unix sockets, since it skips the seccomp filter that otherwise blocks `socket(AF_UNIX, ...)` calls |
| `sandbox.network.allowLocalBinding` | boolean | `false` | Allow binding to localhost ports (macOS) |
| `sandbox.network.allowedDomains` | array | `[]` | Network domain allowlist for sandbox |
| `sandbox.network.deniedDomains` | array | `[]` | Network domain denylist for bash sandbox. Takes precedence over wildcards in `allowedDomains`. Supports glob patterns (e.g., `"*.example.com"`) (v2.1.113) |
| `sandbox.network.httpProxyPort` | number | - | HTTP proxy port 1-65535 (custom proxy) |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 proxy port 1-65535 (custom proxy) |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | Only allow domains in managed allowlist (managed settings) |
| `sandbox.network.allowMachLookup` | array | `[]` | (macOS only) Additional XPC/Mach service names the sandbox may look up. Supports a single trailing `*` for prefix matching. Needed for tools that communicate via XPC such as the iOS Simulator or Playwright. Example: `["com.apple.coresimulator.*"]` |
| `sandbox.filesystem.allowWrite` | array | `[]` | Additional paths where sandboxed commands can write. Arrays are merged across all settings scopes. Also merged with paths from `Edit(...)` allow permission rules. Prefix: `/` (absolute), `~/` (home), `./` or none (project-relative in project settings, `~/.claude`-relative in user settings). The older `//` prefix for absolute paths still works. **Note:** This differs from [Read/Edit permission rules](#tool-permission-syntax), which use `//` for absolute and `/` for project-relative |
| `sandbox.filesystem.denyWrite` | array | `[]` | Paths where sandboxed commands cannot write. Arrays are merged across all settings scopes. Also merged with paths from `Edit(...)` deny permission rules. Same path prefix conventions as `allowWrite` |
| `sandbox.filesystem.denyRead` | array | `[]` | Paths where sandboxed commands cannot read. Arrays are merged across all settings scopes. Also merged with paths from `Read(...)` deny permission rules. Same path prefix conventions as `allowWrite` |
| `sandbox.filesystem.allowRead` | array | `[]` | Paths to re-allow read access within `denyRead` regions. Takes precedence over `denyRead`. Arrays are merged across all settings scopes. Same path prefix conventions as `allowWrite` |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **(Managed only)** Only `allowRead` paths from managed settings are respected. `allowRead` entries from user, project, and local settings are ignored |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | (macOS only) Allow access to system TLS trust (`com.apple.trustd.agent`); reduces security |
| `sandbox.bwrapPath` | string | - | **（Linux/WSL 仅限 managed）** 用于创建沙盒的 `bwrap`（bubblewrap）二进制的自定义路径。当 `bwrap` 安装在非标准位置或通过 managed 镜像分发时使用。仅在 managed 设置中生效 (v2.1.133) |
| `sandbox.socatPath` | string | - | **（Linux/WSL 仅限 managed）** 沙盒网络代理使用的 `socat` 二进制的自定义路径。当 `socat` 安装在非标准位置或通过 managed 镜像分发时使用。仅在 managed 设置中生效 (v2.1.133) |

**Example:**
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker", "gh"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

---

## Plugins

Configure Claude Code plugins and marketplaces.

### Plugin Settings

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enabledPlugins` | object | Any | Enable/disable specific plugins |
| `extraKnownMarketplaces` | object | Project | Add custom plugin marketplaces (team sharing via `.claude/settings.json`) |
| `strictKnownMarketplaces` | array | Managed only | Allowlist of permitted marketplaces |
| `strictPluginOnlyCustomization` | boolean \| array | 仅托管 | 阻止来自用户和项目来源的技能、Agent、hooks 和 MCP 服务器，使其只能来自插件或托管设置。`true` 锁定全部四个表面；数组如 `["skills", "hooks"]` 仅锁定指定表面 |
| `skippedMarketplaces` | array | Any | Marketplaces user declined to install *(in JSON schema, not on official settings page)* |
| `skippedPlugins` | array | Any | Plugins user declined to install *(in JSON schema, not on official settings page)* |
| `pluginConfigs` | object | Any | Per-plugin MCP server configs (keyed by `plugin@marketplace`) *(in JSON schema, not on official settings page)* |
| `blockedMarketplaces` | array | Managed only | Block specific plugin marketplaces. Each entry can match by source string, `hostPattern`, or `pathPattern` — as of v2.1.119 the `hostPattern` and `pathPattern` matchers are correctly enforced before any download touches the filesystem, so blocked marketplaces never reach disk |
| `pluginTrustMessage` | string | Managed only | Custom message displayed when prompting users to trust plugins |

**Marketplace source types:** `github`, `git`, `directory`, `hostPattern`, `settings`, `url`, `npm`, `file`. Use `source: 'settings'` to declare a small set of plugins inline without setting up a hosted marketplace repository.

**Example:**
```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "experimental@acme-tools": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "inline-tools": {
      "source": {
        "source": "settings",
        "name": "inline-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": { "source": "github", "repo": "acme-corp/code-formatter" }
          }
        ]
      }
    }
  }
}
```

---

## Model Configuration

### Model Aliases

| Alias | Description |
|-------|-------------|
| `"default"` | Recommended for your account type |
| `"sonnet"` | Latest Sonnet model (Claude Sonnet 4.6（Anthropic API；第三方提供商为 4.5）) |
| `"opus"` | Latest Opus model (Claude Opus 4.7（Anthropic API；Bedrock/Vertex/Foundry 为 4.6）。自 v2.1.142 起也是快速模式默认值) |
| `"haiku"` | Fast Haiku model |
| `"sonnet[1m]"` | Sonnet with 1M token context |
| `"opus[1m]"` | Opus with 1M token context (default on Max, Team, and Enterprise since v2.1.75) |
| `"opusplan"` | Opus for planning, Sonnet for execution |

**Example:**
```json
{
  "model": "opus"
}
```

> **注意 (v2.1.144)：** `/model` 仅更改**当前会话**的模型。在 `/model` 选择器中按 `d` 可将选择设为默认。`model` 设置和 `ANTHROPIC_MODEL` 继续控制持久默认值。

### Model Overrides

Map Anthropic model IDs to provider-specific model IDs for Bedrock, Vertex, or Foundry deployments.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `effortLevel` | string | - | Persist the effort level across sessions. Accepts `"low"`, `"medium"`, `"high"`, or `"xhigh"` (Opus 4.7 only, v2.1.111). Written automatically when you run `/effort low`, `/effort medium`, `/effort high`, or `/effort xhigh`. Supported on Opus 4.6, Sonnet 4.6, and Opus 4.7. Unsupported levels fall back to the highest supported level on the active model |
  "maxSkillDescriptionChars": 1536,
  "skillListingBudgetFraction": 0.01,
  "disableAgentView": false,
  "syntaxHighlightingDisabled": false,
| `modelOverrides` | object | - | Map model picker entries to provider-specific IDs (e.g., Bedrock inference profile ARNs). Each key is a model picker entry name, each value is the provider model ID |

**Example:**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### Effort Level

The `/model` command exposes an **effort level** control that adjusts how much reasoning the model applies per response. Use the ← → arrow keys in the `/model` UI to cycle through effort levels.

| Effort Level | Description |
|-------------|-------------|
| Max | Maximum reasoning depth, Opus 4.6 only |
| XHigh | Extended high reasoning depth, Opus 4.7 only (default on Opus 4.7 across all plans, v2.1.111) |
| High (default on Opus 4.6/Sonnet 4.6) | Full reasoning depth, best for complex tasks |
| Medium | Balanced reasoning, good for everyday tasks |
| Low | Minimal reasoning, fastest responses |

**How to use:**
1. Run `/effort low`, `/effort medium`, or `/effort high` to set directly (v2.1.76+)
2. Or run `/model` → select a model → use **← →** arrow keys to adjust
3. The setting persists via the `effortLevel` key in `settings.json`

**Note:** Effort level is available for Opus 4.6, Sonnet 4.6, and Opus 4.7 on Max and Team plans. The default was changed from High to Medium in v2.1.68, then changed back to **High** for API-key, Bedrock/Vertex/Foundry, Team, and Enterprise users in v2.1.94. In v2.1.117, the default was also raised from `medium` to `high` for Pro/Max subscribers on Opus 4.6 and Sonnet 4.6, bringing all tiers into alignment on `high`. v2.1.111 introduced **`xhigh`** (Opus 4.7 only) and made it the default effort level on Opus 4.7 across all plans. As of v2.1.75, 1M context window for Opus 4.6 is available by default on Max, Team, and Enterprise plans.

**Skill template variable (v2.1.120):** Inside skill files, use `${CLAUDE_EFFORT}` to reference the current effort level. Useful for skill instructions that should adapt their depth or detail based on the active effort tier.

### Model Environment Variables

Configure via `env` key:

```json
{
  "env": {
    "ANTHROPIC_MODEL": "sonnet",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "custom-haiku-model",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "custom-sonnet-model",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "custom-opus-model",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku",
    "MAX_THINKING_TOKENS": "10000"
  }
}
```

---

## Display & UX

### Display Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `statusLine` | object | - | Custom status line configuration |
| `outputStyle` | string | `"default"` | Output style (e.g., `"Explanatory"`) |
| `spinnerTipsEnabled` | boolean | `true` | Show tips while waiting |
| `spinnerVerbs` | object | - | Custom spinner verbs with `mode` ("append" or "replace") and `verbs` array |
| `spinnerTipsOverride` | object | - | Custom spinner tips with `tips` (string array) and optional `excludeDefault` (boolean). When `excludeDefault` is `true`, only custom tips show; when `false` or absent, custom tips merge with built-in tips. As of v2.1.121, `excludeDefault: true` also suppresses time-based spinner tips |
| `respectGitignore` | boolean | `true` | Respect .gitignore in file picker |
| `prefersReducedMotion` | boolean | `false` | Reduce animations and motion effects in the UI |
| `syntaxHighlightingDisabled` | boolean | `false` | 禁用 diff、代码块和文件预览中的语法高亮。与 `CLAUDE_CODE_SYNTAX_HIGHLIGHT` 环境变量不同，后者仅控制 diff 输出 |
| `fileSuggestion` | object | - | Custom file suggestion command (see File Suggestion Configuration below) |
| `autoScrollEnabled` | boolean | `true` | Auto-scroll the conversation in fullscreen mode. Set to `false` to disable automatic scrolling (v2.1.110). Versions before v2.1.119 stored this in `~/.claude.json` |
| `editorMode` | string | `"normal"` | Key binding mode for the input prompt: `"normal"` or `"vim"`. Appears in `/config` as **Editor mode**. Versions before v2.1.119 stored this in `~/.claude.json` |
| `showTurnDuration` | boolean | `true` | Show turn duration messages after responses (e.g., "Cooked for 1m 6s"). Versions before v2.1.119 stored this in `~/.claude.json` |
| `teammateMode` | string | `"auto"` | How [agent team](https://code.claude.com/docs/en/agent-teams) teammates display: `"auto"` (picks split panes in tmux or iTerm2, in-process otherwise), `"in-process"`, or `"tmux"`. See [choose a display mode](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode). Versions before v2.1.119 stored this in `~/.claude.json` |
| `terminalProgressBarEnabled` | boolean | `true` | Show the terminal progress bar in supported terminals (ConEmu, Ghostty 1.2.0+, and iTerm2 3.6.6+). Appears in `/config` as **Terminal progress bar**. Versions before v2.1.119 stored this in `~/.claude.json` |
| `preferredNotifChannel` | string | `"auto"` | Method for task-complete and permission-prompt notifications. Values: `"auto"`, `"terminal_bell"`, `"iterm2"`, `"iterm2_with_bell"`, `"kitty"`, `"ghostty"`, `"notifications_disabled"`. Default `"auto"` sends a desktop notification in iTerm2, Ghostty, and Kitty and does nothing in other terminals. Set `"terminal_bell"` to ring the bell character in any terminal. Appears in `/config` as **Notifications**. See [Get a terminal bell or notification](https://code.claude.com/docs/en/terminal-config#get-a-terminal-bell-or-notification) |

### Global Config Settings (`~/.claude.json`)

These IDE-related preferences are stored in `~/.claude.json`, **not** `settings.json`.

> **v2.1.119 migration note:** As of v2.1.119, `autoScrollEnabled`, `editorMode`, `showTurnDuration`, `teammateMode`, and `terminalProgressBarEnabled` moved into `settings.json` and are documented in the Display Settings table above. Earlier versions stored them here.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `autoConnectIde` | boolean | `false` | Automatically connect to a running IDE when Claude Code starts from an external terminal. Appears in `/config` as **Auto-connect to IDE (external terminal)** when running outside a VS Code or JetBrains terminal |
| `autoInstallIdeExtension` | boolean | `true` | Automatically install the Claude Code IDE extension when running from a VS Code terminal. Appears in `/config` as **Auto-install IDE extension**. Can also be disabled via `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` env var |
| `externalEditorContext` | boolean | `false` | 用 `Ctrl+G` 打开外部编辑器时，将 Claude 的上一次回复作为 `#` 注释上下文预填。设为 `true` 启用 |

### Workspace & Teams

| Key | Type | Description |
|-----|------|-------------|
| `sshConfigs` | object[] | SSH connection definitions surfaced as a dropdown in Desktop. Each entry must include `id`, `name`, and `sshHost`; optionally `sshPort`, `sshIdentityFile`, and `startDirectory` |

**Field reference:**

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | Unique identifier for the SSH connection entry |
| `name` | yes | Display name shown in the Desktop dropdown |
| `sshHost` | yes | SSH host (e.g., `user@dev.example.com` or `dev.example.com`) |
| `sshPort` | no | SSH port number |
| `sshIdentityFile` | no | Path to the SSH identity file (private key) |
| `startDirectory` | no | Initial working directory after connecting |

**Example:**
```json
{
  "sshConfigs": [
    {
      "id": "dev-vm",
      "name": "Dev VM",
      "sshHost": "user@dev.example.com",
      "sshPort": 22,
      "sshIdentityFile": "~/.ssh/id_ed25519",
      "startDirectory": "/home/user/project"
    }
  ]
}
```

### Status Line Configuration

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2,
    "refreshInterval": 5
  }
}
```

| Field | Description |
|-------|-------------|
| `type` | Set to `"command"` to run a shell script |
| `command` | Shell command or script path that generates the status line output |
| `padding` | Extra horizontal spacing (in characters) added to status line content. Defaults to `0`. Controls relative indentation beyond the interface's built-in spacing |
| `refreshInterval` | Re-run the command every N seconds in addition to event-driven updates. Minimum is `1`. Useful when the status line shows time-based data (e.g., a clock) or when background subagents change git state while the main session is idle. Leave unset to run only on events (v2.1.97) |

**Status Line Input Fields:**

The status line command receives a JSON object on stdin. For the full JSON schema and examples, see the [Status Line Documentation](https://code.claude.com/docs/en/statusline).

| Field | Description |
|-------|-------------|
| `model.id`, `model.display_name` | Current model identifier and display name |
| `cwd`, `workspace.current_dir` | Current working directory (both contain the same value; `workspace.current_dir` preferred) |
| `workspace.project_dir` | Directory where Claude Code was launched (may differ from `cwd` if working directory changes) |
| `workspace.added_dirs` | Additional directories added via `/add-dir` or `--add-dir` |
| `workspace.git_worktree` | Git worktree name when inside a linked worktree created with `git worktree add`. Absent in the main working tree (v2.1.97) |
| `cost.total_cost_usd` | Total session cost in USD |
| `cost.total_duration_ms` | Total wall-clock time since session started, in milliseconds |
| `cost.total_api_duration_ms` | Total time spent waiting for API responses, in milliseconds |
| `cost.total_lines_added`, `cost.total_lines_removed` | Lines of code changed during the session |
| `context_window.total_input_tokens`, `context_window.total_output_tokens` | Cumulative token counts across the session |
| `context_window.context_window_size` | Maximum context window size in tokens (200000 default, 1000000 for extended context) |
| `context_window.used_percentage` | Pre-calculated percentage of context window used |
| `context_window.remaining_percentage` | Pre-calculated percentage of context window remaining |
| `context_window.current_usage` | Token counts from the last API call (input, output, cache tokens) |
| `exceeds_200k_tokens` | Whether total tokens from the most recent API response exceeds 200k (fixed threshold) |
| `rate_limits.five_hour.used_percentage` | Five-hour rate limit usage percentage (v2.1.80+) |
| `rate_limits.five_hour.resets_at` | Five-hour rate limit reset timestamp (Unix epoch seconds) |
| `rate_limits.seven_day.used_percentage` | Seven-day rate limit usage percentage |
| `rate_limits.seven_day.resets_at` | Seven-day rate limit reset timestamp (Unix epoch seconds) |
| `session_id` | Unique session identifier |
| `session_name` | Custom session name set with `--name` or `/rename`. Absent if no custom name set |
| `transcript_path` | Path to conversation transcript file |
| `version` | Claude Code version |
| `output_style.name` | Name of the current output style |
| `vim.mode` | Current vim mode (`NORMAL` or `INSERT`) when vim mode is enabled |
| `agent.name` | Agent name when running with `--agent` flag or agent settings |
| `effort.level` | Current reasoning effort (`low`, `medium`, `high`, `xhigh`, or `max`). Reflects the live session value, including mid-session `/effort` changes. Absent when the current model does not support the effort parameter (v2.1.121) |
| `thinking.enabled` | Whether extended thinking is enabled for the session (v2.1.121) |
| `worktree.name` | Name of the active worktree (present only during `--worktree` sessions) |
| `worktree.path` | Absolute path to the worktree directory |
| `worktree.branch` | Git branch name for the worktree. Absent for hook-based worktrees |
| `worktree.original_cwd` | Directory before entering the worktree |
| `worktree.original_branch` | Git branch checked out before entering the worktree. Absent for hook-based worktrees |
| `github` | 当前分支检测到时的 GitHub 仓库和 pull-request 信息 — 仓库标识和关联的 PR (v2.1.145) |

### File Suggestion Configuration

The file suggestion script receives a JSON object on stdin (e.g., `{"query": "src/comp"}`) and must output up to 15 file paths (one per line).

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**Example:**
```json
{
  "statusLine": {
    "type": "command",
    "command": "git branch --show-current 2>/dev/null || echo 'no-branch'"
  },
  "spinnerTipsEnabled": true,
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Cooking", "Brewing", "Crafting", "Conjuring"]
  },
  "spinnerTipsOverride": {
    "tips": ["Use /compact at ~50% context", "Start with plan mode for complex tasks"],
    "excludeDefault": true
  }
}
```

---

## AWS & Cloud Credentials

### AWS Settings

| Key | Type | Description |
|-----|------|-------------|
| `awsAuthRefresh` | string | Script to refresh AWS auth (modifies `.aws` dir) |
| `awsCredentialExport` | string | Script outputting JSON with AWS credentials |

**Example:**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| Key | Type | Description |
|-----|------|-------------|
| `otelHeadersHelper` | string | Script to generate dynamic OpenTelemetry headers |

**Example:**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## Environment Variables (via `env`)

Set environment variables for all Claude Code sessions.

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### Common Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key for authentication |
| `ANTHROPIC_AUTH_TOKEN` | OAuth token |
| `CLAUDE_CODE_OAUTH_TOKEN` | OAuth access token for Claude.ai authentication. Alternative to `/login` for SDK and automated environments. Takes precedence over keychain-stored credentials |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | OAuth refresh token for Claude.ai authentication. When set, `claude auth login` exchanges this token directly instead of opening a browser. Requires `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | Space-separated OAuth scopes the refresh token was issued with (e.g., `"user:profile user:inference user:sessions:claude_code"`). Required when `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` is set |
| `ANTHROPIC_WORKSPACE_ID` | [workload identity federation](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation) 的工作区 ID。当联合规则范围超过一个工作区时设置，以便 token 交换知道目标工作区 (v2.1.141) |
| `ANTHROPIC_BASE_URL` | Custom API endpoint |
| `ANTHROPIC_BEDROCK_BASE_URL` | Override Bedrock endpoint URL |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | Override the Bedrock Mantle endpoint URL. See [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock service tier: `default`, `flex`, or `priority`. Sent as the `X-Amzn-Bedrock-Service-Tier` header on every request. See [Amazon Bedrock service tiers](https://code.claude.com/docs/en/amazon-bedrock#service-tiers) (v2.1.122) |
| `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` | Set by host platforms that embed Claude Code and manage model provider routing on the user's behalf. When set, provider-selection / endpoint / authentication env vars in `settings.json` (e.g., `CLAUDE_CODE_USE_BEDROCK`, `ANTHROPIC_BASE_URL`, `ANTHROPIC_API_KEY`) are ignored so user settings cannot override the host's routing. The automatic telemetry opt-out for Bedrock/Vertex/Foundry is also skipped, so telemetry follows the standard `DISABLE_TELEMETRY` opt-out (v2.1.126) |
| `ANTHROPIC_VERTEX_BASE_URL` | Override Vertex AI endpoint URL |
| `ANTHROPIC_BETAS` | Comma-separated Anthropic beta header values |
| `ANTHROPIC_VERTEX_PROJECT_ID` | GCP project ID for Vertex AI |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | Model ID to add as a custom entry in the `/model` picker. Use to make a non-standard or gateway-specific model selectable without replacing built-in aliases |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | Display name for the custom model entry in the `/model` picker. Defaults to the model ID when not set |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | Display description for the custom model entry in the `/model` picker. Defaults to `Custom model (<model-id>)` when not set |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES` | Override capability detection for the custom model entry. Comma-separated values (e.g., `effort,thinking`). Required when the custom model supports features the auto-detection cannot confirm. See [model configuration](https://code.claude.com/docs/en/model-config#customize-pinned-model-display-and-capabilities) |
| `ANTHROPIC_MODEL` | Name of the model to use. Accepts aliases (`sonnet`, `opus`, `haiku`) or full model IDs. Overrides the `model` setting |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Override the Haiku model alias with a custom model ID (e.g., for third-party deployments) |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | Customize the Haiku entry label in the `/model` picker when using a pinned model on Bedrock/Vertex/Foundry. Defaults to the model ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | Customize the Haiku entry description in the `/model` picker. Defaults to `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | Override capability detection for a pinned Haiku model. Comma-separated values (e.g., `effort,thinking`). Required when the pinned model supports features the auto-detection cannot confirm |
| `CLAUDECODE` | Set to `1` in shell environments Claude Code spawns (Bash tool, tmux sessions). Not set in hooks or status line commands. Use to detect when a script is running inside a Claude Code shell |
| `CLAUDE_CODE_SESSION_ID` | 只读。自动设置到 Bash 子进程环境中为当前 Claude Code 会话的 ID。从 Bash 命令、钩子或技能辅助程序中读取，以将日志、指标或遥测数据与特定会话关联，无需解析转储路径 (v2.1.132) |
| `AI_AGENT` | Set automatically by Claude Code in subprocess environments (Bash tool, hooks, MCP stdio servers). Generic flag identifying the parent process as an AI agent — useful for tools that adapt behavior when invoked from any AI agent rather than checking each agent-specific variable like `CLAUDECODE` *(in v2.1.120 changelog, not yet on official env-vars page)* |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | Set to `1` to allow fast mode when the organization status check fails due to a network error. Useful when a corporate proxy blocks the status endpoint |
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock (`1` to enable) |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex AI (`1` to enable) |
| `CLAUDE_CODE_USE_FOUNDRY` | Use Microsoft Foundry (`1` to enable) |
| `CLAUDE_CODE_USE_MANTLE` | Use the Bedrock [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) (`1` to enable) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Set to `1` to enable the PowerShell tool on Windows (opt-in preview). When enabled, Claude can run PowerShell commands natively instead of routing through Git Bash. Only supported on native Windows, not WSL (v2.1.84) |
| `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` | 设为 `1` 时，Claude Code 在生成 PowerShell 用于工具调用、钩子和状态行命令时不再传递 `-ExecutionPolicy Bypass`，而是遵循机器的有效执行策略。默认情况下 Claude Code 在进程范围内绕过执行策略，以便 `.ps1` 脚本和模块导入在默认 Restricted 的 Windows 上正常工作。从不覆盖组策略 `MachinePolicy`/`UserPolicy` (v2.1.143) |
| `CLAUDE_CODE_REMOTE` | Read-only. Set automatically to `true` when Claude Code is running as a cloud session. Read this from a hook or setup script to detect whether you are in a cloud environment |
| `CLAUDE_CODE_REMOTE_SESSION_ID` | Read-only. Set automatically in cloud sessions to the current session's ID. Read this to construct a link back to the session transcript |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | Prefix for auto-generated Remote Control session names. Defaults to the machine hostname |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Enable/disable telemetry (`0` or `1`) |
| `DISABLE_ERROR_REPORTING` | Disable error reporting (`1` to disable) |
| `DISABLE_AUTOUPDATER` | Set to `1` to disable automatic update checks against the npm registry. Also configurable as a startup-only var — see [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) |
| `DISABLE_UPDATES` | Set to `1` to completely block all update paths — automatic checks, notifications, and manual `claude update`. Stricter than `DISABLE_AUTOUPDATER`, which only disables the background check. Use in environments where all updates must be blocked until explicitly re-enabled *(in v2.1.118 changelog, not yet on official env-vars page)* |
| `DISABLE_TELEMETRY` | Disable telemetry (`1` to disable) |
| `MCP_TIMEOUT` | MCP startup timeout in ms |
| `MAX_MCP_OUTPUT_TOKENS` | Max MCP output tokens (default: 25000). Warning displayed when output exceeds 10,000 tokens |
| `API_TIMEOUT_MS` | Timeout in ms for API requests (default: 600000) |
| `BASH_MAX_TIMEOUT_MS` | Bash command timeout |
| `BASH_MAX_OUTPUT_LENGTH` | Max bash output length |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Auto-compact threshold percentage (1-100). Default is ~95%. Set lower (e.g., `50`) to trigger compaction earlier. Values above 95% have no effect. Use `/context` to monitor current usage. Example: `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | Override the context window size Claude Code assumes for the active model. Only takes effect when `DISABLE_COMPACT` is also set. Use when routing to a model through `ANTHROPIC_BASE_URL` whose context window does not match the built-in size for its name |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Keep cwd between bash calls (`1` to enable) |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Disable background tasks (`1` to disable) |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | 设为 `1` 可关闭后台 Agent 和 Agent 视图（`claude agents`、`--bg`、`/background`、按需监督器）。`disableAgentView` 设置的环境变量等效项 *（在官方设置页面引用；未在环境变量页面列出）* |
| `ENABLE_TOOL_SEARCH` | MCP tool search threshold (e.g., `auto:5`) |
| `ENABLE_PROMPT_CACHING_1H` | Opt into 1-hour prompt cache TTL. Replaces the deprecated `ENABLE_PROMPT_CACHING_1H_BEDROCK` *(in v2.1.108 changelog, not yet on official env-vars page)* |
| `FORCE_PROMPT_CACHING_5M` | Force 5-minute prompt cache TTL *(in v2.1.108 changelog, not yet on official env-vars page)* |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | Opt out of away summary / idle-session recap. Set to `0` to disable. Pairs with the `awaySummaryEnabled` setting (v2.1.110) |
| `DISABLE_PROMPT_CACHING` | Disable all prompt caching (`1` to disable) |
| `DISABLE_PROMPT_CACHING_HAIKU` | Disable Haiku prompt caching |
| `DISABLE_PROMPT_CACHING_SONNET` | Disable Sonnet prompt caching |
| `DISABLE_PROMPT_CACHING_OPUS` | Disable Opus prompt caching |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | Request 1-hour cache TTL on Bedrock (`1` to enable) *(not in official docs — unverified; v2.1.108 changelog says deprecated, replaced by `ENABLE_PROMPT_CACHING_1H`)* |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | Disable experimental beta features (`1` to disable) |
| `CLAUDE_CODE_SHELL` | Override automatic shell detection |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | Override default file read token limit |
| `CLAUDE_CODE_GLOB_HIDDEN` | Set to `false` to exclude dotfiles from results when Claude invokes the Glob tool. Included by default. Does not affect `@` file autocomplete, `ls`, Grep, or Read |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | Set to `false` to make the Glob tool respect `.gitignore` patterns. By default, Glob returns all matching files including gitignored ones. Does not affect `@` file autocomplete, which has its own `respectGitignore` setting |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Timeout in seconds for Glob file discovery |
| `CLAUDE_CODE_ENABLE_TASKS` | Set to `true` to enable task tracking in non-interactive mode (`-p` flag). Tasks are on by default in interactive mode |
| `CLAUDE_CODE_SIMPLE` | Set to `1` to run with a minimal system prompt and only the Bash, file read, and file edit tools. Also configurable as a startup-only var — see [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | Auto-exit SDK mode after idle duration (ms) |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | Disable adaptive thinking (`1` to disable) |
| `CLAUDE_CODE_DISABLE_THINKING` | Force-disable extended thinking (`1` to disable) |
| `DISABLE_INTERLEAVED_THINKING` | Prevent interleaved-thinking beta header from being sent (`1` to disable) |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | Disable 1M token context window (`1` to disable) |
| `CLAUDE_CODE_ACCOUNT_UUID` | Override account UUID for authentication |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | Disable git-related system prompt instructions |
| `CLAUDE_CODE_NEW_INIT` | Set to `true` to make `/init` run an interactive setup flow. Asks which files to generate (CLAUDE.md, skills, hooks) before exploring the codebase. Without this, `/init` generates a CLAUDE.md automatically |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | Path to one or more read-only plugin seed directories, separated by `:` on Unix or `;` on Windows. Bundle pre-populated plugins into a container image. Claude Code registers marketplaces from these directories at startup and uses pre-cached plugins without re-cloning |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | Enable Claude.ai MCP servers |
| `CLAUDE_CODE_EFFORT_LEVEL` | Set effort level: `low`, `medium`, `high`, `xhigh` (Opus 4.7 only, v2.1.111), `max` (Opus 4.6 only), or `auto` (use model default). Takes precedence over `/effort` and the `effortLevel` setting. Also configurable as a startup-only var — see [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_EFFORT` | 只读。注入到 Bash 工具子进程和钩子处理器中，携带当前激活的努力级别，以便 shell 脚本和钩子适配当前层级（`CLAUDE_CODE_EFFORT_LEVEL` 的配套变量；v2.1.133）。在 skill 文件中使用 `${CLAUDE_EFFORT}` *（在 changelog 中，不在官方 env-vars 页面 — 只读，不可用户配置）* |
| `CLAUDE_CODE_MAX_TURNS` | Maximum agentic turns before stopping *(not in official docs — unverified)* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Equivalent of setting `DISABLE_AUTOUPDATER`, `DISABLE_FEEDBACK_COMMAND`, `DISABLE_ERROR_REPORTING`, and `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | Skip first-run settings setup flow *(not in official docs — unverified)* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | Override prompt caching behavior *(not in official docs — unverified)* |
| `CLAUDE_CODE_DISABLE_TOOLS` | Comma-separated list of tools to disable *(not in official docs — unverified)* |
| `CLAUDE_CODE_DISABLE_MCP` | Disable all MCP servers (`1` to disable) *(not in official docs — unverified)* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens per response. Default: 32,000 (64,000 for Opus 4.6 as of v2.1.77). Upper bound: 64,000 (128,000 for Opus 4.6 and Sonnet 4.6 as of v2.1.77) |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | Disable fast mode entirely (`1` to disable) |
| `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` | **v2.1.160 已移除** — 此环境变量现在是无操作（no-op）。无论是否设置此变量，fast mode 都运行在默认模型上。此前用于将 [fast mode](https://code.claude.com/docs/en/fast-mode) 固定为 Claude Opus 4.6 而非默认 (v2.1.142–v2.1.159) |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | Set to `1` to disable the non-streaming fallback when a streaming request fails mid-stream. Streaming errors propagate to the retry layer instead. Useful when a proxy or gateway causes the fallback to produce duplicate tool execution (v2.1.83) |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | Abort stalled streams (`1` to enable) |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | Enable fine-grained tool streaming (`1` to enable) |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Disable auto memory (`1` to disable) |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | Disable file checkpointing for `/rewind` (`1` to disable) |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | Disable attachment processing (`1` to disable) |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | Prevent loading CLAUDE.md files (`1` to disable) |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | Auto-resume if previous session ended mid-turn (`1` to enable) |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | Set to `1` to skip writing prompt history and session transcripts to disk. Sessions started with this variable set do not appear in `--resume`, `--continue`, or up-arrow history. Useful for ephemeral scripted sessions |
| `CLAUDE_CODE_USER_EMAIL` | Provide user email synchronously for authentication |
| `CLAUDE_CODE_ORGANIZATION_UUID` | Provide organization UUID synchronously for authentication |
| `CLAUDE_CONFIG_DIR` | Custom config directory (overrides default `~/.claude`) |
| `CLAUDE_CODE_TMPDIR` | Override the temp directory used for internal temp files. Claude Code appends `/claude/` to this path. Default: `/tmp` on Unix/macOS, `os.tmpdir()` on Windows |
| `ANTHROPIC_CUSTOM_HEADERS` | Custom headers for API requests (`Name: Value` format, newline-separated for multiple headers) |
| `ANTHROPIC_FOUNDRY_API_KEY` | API key for Microsoft Foundry authentication |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Base URL for Foundry resource |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Foundry resource name |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock API key for authentication |
| `ANTHROPIC_SMALL_FAST_MODEL` | **DEPRECATED** — Use `ANTHROPIC_DEFAULT_HAIKU_MODEL` instead |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | AWS region for deprecated Haiku-class model override |
| `CLAUDE_CODE_SHELL_PREFIX` | Command prefix prepended to bash commands |
| `BASH_DEFAULT_TIMEOUT_MS` | Default bash command timeout in ms |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Skip AWS auth for Bedrock (`1` to skip) |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | Skip Azure auth for Foundry (`1` to skip) |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | Skip AWS authentication for Bedrock Mantle (e.g., when using an LLM gateway) |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Skip Google auth for Vertex (`1` to skip) |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | Allow proxy to perform DNS resolution |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | Credential refresh interval in ms for `apiKeyHelper` |
| `CLAUDE_CODE_CLIENT_CERT` | Client certificate path for mTLS |
| `CLAUDE_CODE_CLIENT_KEY` | Client private key path for mTLS |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | Passphrase for encrypted mTLS key |
| `CLAUDE_CODE_CERT_STORE` | Comma-separated list of CA certificate sources for TLS connections: `bundled` (Mozilla CA set shipped with Claude Code) and/or `system` (OS trust store). Default: `bundled,system`. The native binary distribution is required for system store integration; on the Node.js runtime, only the bundled set is used regardless of this value (v2.1.101) |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | Plugin marketplace git clone timeout in ms (default: 120000) |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | 设为 `1` 时通过 HTTPS（而非 SSH）克隆 GitHub `owner/repo` 简写插件源。适用于插件安装/更新和 `/plugin marketplace add`/`update`。在没有为 `github.com` 配置 SSH 密钥的 CI 运行器或容器中很有用 (v2.1.141) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | Override the plugins root directory |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | Skip auto-adding the official marketplace (`1` to disable) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | Wait for plugin install to complete before first query (`1` to enable) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | Timeout in ms for synchronous plugin install |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | Set to `1` to keep the existing marketplace cache when a `git pull` fails instead of wiping and re-cloning. Useful in offline or airgapped environments where re-cloning would fail the same way |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | Hide email/org info from UI *(not in official docs — unverified)* |
| `CLAUDE_CODE_DISABLE_CRON` | Disable scheduled/cron tasks (`1` to disable) |
| `DISABLE_INSTALLATION_CHECKS` | Disable installation warnings |
| `DISABLE_FEEDBACK_COMMAND` | Disable the `/feedback` command. The older name `DISABLE_BUG_COMMAND` is also accepted |
| `DISABLE_DOCTOR_COMMAND` | Hide the `/doctor` command (`1` to disable) |
| `DISABLE_LOGIN_COMMAND` | Hide the `/login` command (`1` to disable) |
| `DISABLE_LOGOUT_COMMAND` | Hide the `/logout` command (`1` to disable) |
| `DISABLE_UPGRADE_COMMAND` | Hide the `/upgrade` command (`1` to disable) |
| `DISABLE_EXTRA_USAGE_COMMAND` | 隐藏 `/extra-usage` 命令 — v2.1.144 中重命名为 `/usage-credits`，但此环境变量名称保持不变（`1` 以禁用） |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | Hide the `/install-github-app` command (`1` to disable) |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | Disable flavor text and non-essential model calls *(not in official docs — unverified)* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | Override debug log file directory path |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | Minimum debug log level |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | Force auto-backgrounding of long tasks (`1` to enable) |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | Prevent remapping Opus 4.0/4.1 to newer models (`1` to disable) |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | Trigger fallback model for all primary models, not just default (`1` to enable) |
| `CCR_FORCE_BUNDLE` | Set to `1` to force `claude --remote` to bundle and upload your local repository even when GitHub access is available. Also configurable as a startup-only var — see [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_GIT_BASH_PATH` | Windows only: path to the Git Bash executable (`bash.exe`). Use when Git Bash is installed but not in your PATH |
| `DISABLE_COST_WARNINGS` | Disable cost warning messages |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Override model for subagents (e.g., `haiku`, `sonnet`) |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Set to `1` to strip Anthropic and cloud provider credentials from subprocess environments (Bash tool, hooks, MCP stdio servers). Use for defense-in-depth when subprocesses should not inherit API keys (v2.1.83) |
| `CLAUDE_CODE_SCRIPT_CAPS` | JSON object limiting how many times specific scripts may be invoked per session when `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` is set. Keys are substrings matched against the command text; values are integer call limits. For example, `{"deploy.sh": 2}` allows `deploy.sh` to be called at most twice. Matching is substring-based; runtime fan-out via `xargs` or `find -exec` is not detected — this is a defense-in-depth control |
| `CLAUDE_CODE_PERFORCE_MODE` | Set to `1` to enable Perforce-aware write protection. When set, Edit, Write, and NotebookEdit fail with a `p4 edit <file>` hint if the target file lacks the owner-write bit, which Perforce clears on synced files until `p4 edit` opens them. Prevents Claude Code from bypassing Perforce change tracking (v2.1.98) |
| `CLAUDE_CODE_MAX_RETRIES` | Override API request retry count (default: 10) |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | Max parallel read-only tools (default: 10) |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | Disable built-in subagent types in SDK mode (`1` to disable) |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | Skip `mcp__<server>__` prefix for MCP tools in SDK mode (`1` to enable) |
| `MCP_CONNECTION_NONBLOCKING` | Set to `true` in `-p` mode to skip the MCP connection wait entirely. Bounds `--mcp-config` server connections at 5s instead of blocking on the slowest server *(in v2.1.89 changelog, not yet on official env-vars page)* |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hook timeout in ms (replaces hard 1.5s limit) |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | Disable feedback survey prompts (`1` to disable) |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | Disable terminal title updates (`1` to disable) |
| `CLAUDE_CODE_TMUX_TRUECOLOR` | Set to `1` to allow 24-bit truecolor output inside tmux. By default, Claude Code clamps to 256 colors when `$TMUX` is set because tmux does not pass through truecolor escape sequences unless configured to. Set this after adding `set -ga terminal-overrides ',*:Tc'` to your `~/.tmux.conf` |
| `CLAUDE_CODE_NO_FLICKER` | Set to `1` to enable flicker-free alt-screen rendering. Eliminates visual flicker during fullscreen redraws (v2.1.88) |
| `CLAUDE_CODE_SCROLL_SPEED` | Mouse wheel scroll multiplier for fullscreen rendering. Increase for faster scrolling, decrease for finer control |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | Set to `1` to disable virtual scrolling in fullscreen rendering and render every message in the transcript. Use if scrolling in fullscreen mode shows blank regions where messages should appear |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | 设为 `1` 完全退出 alternate-screen（全屏）渲染器并使用经典滚动回渲染器。当终端复用器、录制工具或辅助功能工具不能干净地处理 alt-screen 缓冲区时有用 (v2.1.132) |
| `CLAUDE_CODE_DISABLE_MOUSE` | Set to `1` to disable mouse tracking in fullscreen rendering. Useful when mouse events interfere with terminal multiplexers or accessibility tools |
| `CLAUDE_CODE_HIDE_CWD` | Set to `1` to hide the current working directory in the Claude Code startup logo banner. Useful in screen recordings, demos, or shared sessions where the CWD path leaks information about the host or project layout (v2.1.119) |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | 设为 `1` 强制 Claude Code 写入终端的输出同步刷新。默认为异步/缓冲输出以提高性能。当终端输出与子进程输出交错或乱序时用作调试辅助 (v2.1.129) |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | 控制基于包管理器的 Claude Code 后台自动更新检查。设为 `0` 禁用后台检查（Claude Code 不会轮询包管理器获取更新版本）；设为 `1`（默认）保持后台检查启用。独立于 `DISABLE_AUTOUPDATER`，后者控制 npm 注册表自动更新器 (v2.1.129) |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | 设为 `1` 选择从配置的 LLM 网关获取可用模型列表（例如 Bedrock/Vertex 前面的企业代理）。启用后，`/model` 选择器从网关的发现端点填充，而非内置别名列表。当你的网关暴露用户应该选择的精选模型子集时使用 (v2.1.129) |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | 设为 `1` 重新启用面向 OpenTelemetry 企业的会话内质量调查。配置 `OTEL_*` 环境变量或 `feedbackSurveyRate` 时默认抑制调查，以避免将调查数据泄漏到企业遥测管道中。当管理员希望采样调查数据尽管部署了 OTel 时使用 (v2.1.136) |
| `CLAUDE_CODE_ACCESSIBILITY` | Set to `1` to keep native terminal cursor visible for screen readers and accessibility tools |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | Set to `0` to disable syntax highlighting in diff output |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | Skip automatic IDE extension installation (`1` to skip) |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | Override auto IDE connection behavior |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | Override IDE host address for connection |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | Skip IDE lockfile validation (`1` to skip) |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | Debounce interval in ms for OTel headers helper script |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | Timeout in ms for OpenTelemetry flush |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | Timeout in ms for OpenTelemetry shutdown |
| `CLAUDE_ENABLE_BYTE_WATCHDOG` | Set to `1` to force-enable the byte-level streaming idle watchdog, or `0` to force-disable it. When unset, the watchdog is enabled by default for Anthropic API connections. The byte watchdog aborts a connection when no bytes arrive on the wire for the duration set by `CLAUDE_STREAM_IDLE_TIMEOUT_MS` (minimum 5 minutes), independent of the event-level watchdog |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | Timeout in ms for the streaming idle watchdog. Two watchdogs apply: **byte-level** (default and minimum `300000` / 5 minutes, aborts when no bytes arrive on the wire) and **event-level** (default `90000` / 90 seconds, no minimum, aborts when no SSE events arrive). The byte watchdog is enabled by default for Anthropic API connections; control it via `CLAUDE_ENABLE_BYTE_WATCHDOG`. Increase the event timeout if long-running tools or slow networks cause premature timeout errors |
| `OTEL_LOG_TOOL_DETAILS` | Set to `1` to include `tool_parameters` in OpenTelemetry events. Omitted by default for privacy *(in v2.1.85 changelog, not yet on official env-vars page)* |
| `OTEL_LOG_RAW_API_BODIES` | Set to `1` to emit full API request and response bodies as OpenTelemetry log events. Omitted by default for privacy and payload size. Useful for debugging at a gateway or proxy *(in v2.1.111 changelog, not yet on official env-vars page)* |
| `OTEL_LOG_USER_PROMPTS` | Set to `1` to include the `user_system_prompt` field in OpenTelemetry LLM request spans. Omitted by default for privacy — user prompts can contain sensitive data, so opt in only when you control the OTel collector and have policies in place *(in v2.1.121 changelog, not yet on official env-vars page)* |
| `CLAUDE_CODE_FORK_SUBAGENT` | Set to `1` to enable forked subagents on external builds (non-Anthropic-signed distributions). Forked subagents run in an isolated child process instead of sharing the main agent's context *(in v2.1.117 changelog, not yet on official env-vars page)* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | Name of the MCP server, passed as an environment variable to `headersHelper` scripts so they can generate server-specific authentication headers *(in v2.1.85 changelog, not yet on official env-vars page)* |
| `CLAUDE_CODE_MCP_SERVER_URL` | URL of the MCP server, passed as an environment variable to `headersHelper` scripts alongside `CLAUDE_CODE_MCP_SERVER_NAME` *(in v2.1.85 changelog, not yet on official env-vars page)* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Override Opus model alias (e.g., `claude-opus-4-6[1m]`) |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | Customize the Opus entry label in the `/model` picker when using a pinned model on Bedrock/Vertex/Foundry. Defaults to the model ID |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | Customize the Opus entry description in the `/model` picker. Defaults to `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | Override capability detection for a pinned Opus model. Comma-separated values (e.g., `effort,thinking`). Required when the pinned model supports features the auto-detection cannot confirm |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Override Sonnet model alias (e.g., `claude-sonnet-4-6`) |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | Customize the Sonnet entry label in the `/model` picker when using a pinned model on Bedrock/Vertex/Foundry. Defaults to the model ID |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | Customize the Sonnet entry description in the `/model` picker. Defaults to `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | Override capability detection for a pinned Sonnet model. Comma-separated values (e.g., `effort,thinking`). Required when the pinned model supports features the auto-detection cannot confirm |
| `MAX_THINKING_TOKENS` | Maximum extended thinking tokens per response |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Set the context capacity in tokens used for auto-compaction calculations. Defaults to the model's context window (200K standard, 1M for extended context models). Use a lower value (e.g., `500000`) on a 1M model to treat it as 500K for compaction. Capped at actual context window. `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` is applied as a percentage of this value. Setting this decouples the compaction threshold from the status line's `used_percentage` |
| `DISABLE_AUTO_COMPACT` | Disable automatic context compaction (`1` to disable). Manual `/compact` still works |
| `DISABLE_COMPACT` | Disable all compaction — both automatic and manual (`1` to disable) |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | Enable prompt suggestions |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | Require plan mode for sessions |
| `CLAUDE_CODE_TEAM_NAME` | Team name for agent teams |
| `CLAUDE_CODE_TASK_LIST_ID` | Task list ID for task integration |
| `CLAUDE_ENV_FILE` | Custom environment file path |
| `FORCE_AUTOUPDATE_PLUGINS` | Force plugin auto-updates (`1` to enable) |
| `HTTP_PROXY` | HTTP proxy URL for network requests |
| `HTTPS_PROXY` | HTTPS proxy URL for network requests |
| `NO_PROXY` | Comma-separated list of hosts that bypass proxy |
| `MCP_TOOL_TIMEOUT` | MCP tool execution timeout in ms |
| `MCP_CLIENT_SECRET` | MCP OAuth client secret |
| `MCP_OAUTH_CALLBACK_PORT` | MCP OAuth callback port |
| `IS_DEMO` | Enable demo mode |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Character budget for slash command tool output |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Vertex AI region override for Claude 3.5 Haiku |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Vertex AI region override for Claude 3.7 Sonnet |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Vertex AI region override for Claude 4.0 Opus |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Vertex AI region override for Claude 4.0 Sonnet |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Vertex AI region override for Claude 4.1 Opus |

---

## Useful Commands

| Command | Description |
|---------|-------------|
| `/model` | Switch models and adjust Opus 4.6 effort level |
| `/effort` | Set effort level directly: `low`, `medium`, `high`, `xhigh` (Opus 4.7 only, v2.1.111), or `max` (Opus 4.6 only) (v2.1.76+) |
| `/config` | Interactive configuration UI |
| `/memory` | View/edit all memory files |
| `/agents` | Manage subagents |
| `/mcp` | Manage MCP servers |
| `/hooks` | View configured hooks |
| `/plugin` | Manage plugins |
| `claude plugin tag` | Tag a plugin version in a marketplace for distribution. Run from the marketplace repo with the plugin name and version (v2.1.118) |
| `/keybindings` | Configure custom keyboard shortcuts |
| `/skills` | View and manage skills |
| `/permissions` | View and manage permission rules |
| `/usage-credits` | 查看剩余用量额度和限制。v2.1.144 从 `/extra-usage` 重命名（旧名称仍可用） |
| `--doctor` | Diagnose configuration issues |
| `--debug` | Debug mode with hook execution details |

---

## Quick Reference: Complete Example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "agent": "code-reviewer",
  "language": "english",
  "cleanupPeriodDays": 30,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "viewMode": "default",
  "tui": "fullscreen",
  "awaySummaryEnabled": false,
  "includeGitInstructions": true,
  "defaultShell": "bash",
  "plansDirectory": "./plans",
  "claudeMdExcludes": ["**/vendor/**/CLAUDE.md"],
  "effortLevel": "xhigh",

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"],
    "baseRef": "fresh",
    "bgIsolation": "worktree"
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ],
    "soft_deny": ["$defaults", "Never run terraform apply"]
  },

  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*",
      "Agent(*)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../shared/"],
    "defaultMode": "acceptEdits"
  },

  "enableAllProjectMcpServers": true,

  "mcpServers": {
    "always-on-server": {
      "type": "http",
      "url": "https://mcp.example.com",
      "alwaysLoad": true
    }
  },

  "sshConfigs": [
    {
      "id": "dev-vm",
      "name": "Dev VM",
      "sshHost": "user@dev.example.com"
    }
  ],

  "sandbox": {
    "enabled": true,
    "excludedCommands": ["git", "docker"],
    "filesystem": {
      "denyRead": ["./secrets/"],
      "denyWrite": ["./.env"]
    }
  },

  "attribution": {
    "commit": "Generated with Claude Code",
    "pr": ""
  },
  "prUrlTemplate": "https://gitlab.example.com/{owner}/{repo}/-/merge_requests/{number}",

  "statusLine": {
    "type": "command",
    "command": "git branch --show-current"
  },

  "spinnerTipsEnabled": true,
  "spinnerTipsOverride": {
    "tips": ["Custom tip 1", "Custom tip 2"],
    "excludeDefault": false
  },
  "prefersReducedMotion": false,
  "preferredNotifChannel": "terminal_bell",

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium",
    "ANTHROPIC_BEDROCK_SERVICE_TIER": "priority"
  }
}
```

---

## Sources

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code Settings JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub Settings Examples](https://github.com/feiskyer/claude-code-settings)
- [Shipyard - Claude Code CLI Cheatsheet](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Claude Code Environment Variables Reference](https://code.claude.com/docs/en/env-vars)
- [Claude Code Permissions Reference](https://code.claude.com/docs/en/permissions)

<!-- TRANSLATED: Auto-translated from source -->

# Claude Code Settings Reference

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2028%2C%202026%206%3A10%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.86-blue?style=flat&labelColor=555)

Claude Code `settings.json` 文件中所有可用配置选项的综合指南。截至 v2.1.86，Claude Code 公开了 **60+ settings** 和 **100+ environment variables**（使用 `settings.json` 中的 `"env"` field 以避免 wrapper scripts）。

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

Settings 按优先级顺序应用（从高到低）：

| Priority | Location | Scope | Shared? | Purpose |
|----------|----------|-------|---------|---------|
| 1 | Managed settings | Organization | Yes（由 IT 部署） | 无法覆盖的安全策略 |
| 2 | Command line arguments | Session | N/A | 临时单会话覆盖 |
| 3 | `.claude/settings.local.json` | Project | No（git-ignored） | 个人项目特定设置 |
| 4 | `.claude/settings.json` | Project | Yes（committed） | 团队共享设置 |
| 5 | `~/.claude/settings.json` | User | N/A | 全局个人默认设置 |

**Managed settings** 是 organization-enforced 且无法被任何其他级别覆盖，包括 command line arguments。Delivery methods：
- **Server-managed** settings（remote delivery）
- **MDM profiles** — macOS plist 位于 `com.anthropic.claudecode`
- **Registry policies** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode`（admin）和 `HKCU\SOFTWARE\Policies\ClaudeCode`（user-level，最低 policy priority）
- **File** — `managed-settings.json`（macOS: `/Library/Application Support/ClaudeCode/`, Linux/WSL: `/etc/claude-code/`, Windows: `C:\Program Files\ClaudeCode\`）
- **Drop-in directory** — `managed-settings.d/` 与 `managed-settings.json` 并列用于独立的 policy fragments（v2.1.83）。遵循 systemd convention，`managed-settings.json` 首先作为 base 合并，然后 drop-in directory 中的所有 `*.json` 文件按字母顺序排序并合并到上面。后面的文件覆盖前面的文件的 scalar values；arrays 被连接和去重；objects 被 deep-merged。以 `.` 开头的 hidden files 被忽略。使用 numeric prefixes 控制 merge order（例如 `10-telemetry.json`, `20-security.json`）

在 managed tier 内，precedence 是：server-managed > MDM/OS-level policies > file-based（`managed-settings.d/*.json` + `managed-settings.json`）> HKCU registry（仅 Windows）。只使用一个 managed source；sources 不在 tiers 之间合并。在 file-based tier 内，drop-in files 和 base file 一起合并。

> **Note:** 截至 v2.1.75，已弃用的 Windows fallback path `C:\ProgramData\ClaudeCode\managed-settings.json` 已被移除。改用 `C:\Program Files\ClaudeCode\managed-settings.json`。

**Important**:
- `deny` rules 具有最高安全优先级且无法被较低优先级的 allow/ask rules 覆盖。
- Managed settings 可能锁定或覆盖 local behavior，即使 local files 指定了不同的值。
- Array settings（例如 `permissions.allow`）在 scopes 之间**连接和去重** — 来自所有级别的 entries 被组合而非替换。

---

## Core Configuration

### General Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `$schema` | string | - | JSON Schema URL 用于 IDE validation 和 autocompletion（例如 `"https://json.schemastore.org/claude-code-settings.json"`） |
| `model` | string | `"default"` | 覆盖 default model。接受 aliases（`sonnet`, `opus`, `haiku`）或 full model IDs |
| `agent` | string | - | 为 main conversation 设置 default agent。值是来自 `.claude/agents/` 的 agent name。也可通过 `--agent` CLI flag 使用 |
| `language` | string | `"english"` | Claude 的首选 response language |
| `cleanupPeriodDays` | number | `30` | 启动时删除超过此时间未活动的 sessions。设置为 `0` 删除所有现有 transcripts 并完全禁用 session persistence（不写入 `.jsonl` 文件，`/resume` 不显示 conversations，hooks 接收空的 `transcript_path`） |
| `autoUpdatesChannel` | string | `"latest"` | Release channel：`"stable"` 或 `"latest"` |
| `alwaysThinkingEnabled` | boolean | `false` | 为所有 sessions 默认启用 extended thinking |
| `skipWebFetchPreflight` | boolean | `false` | 在 fetching URLs 之前跳过 WebFetch blocklist check *（在 JSON schema 中，不在 official settings page 上）* |
| `availableModels` | array | - | 限制用户可通过 `/model`, `--model`, Config tool, 或 `ANTHROPIC_MODEL` 选择的 models。不影响 Default option。示例：`["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | 要求用户每 session opt in 到 fast mode |
| `defaultShell` | string | `"bash"` | input-box `!` commands 的 Default shell。接受 `"bash"`（默认）或 `"powershell"`。设置 `"powershell"` 在 Windows 上将 interactive `!` commands 路由通过 PowerShell。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`（v2.1.84） |
| `includeGitInstructions` | boolean | `true` | 在 system prompt 中包含 git-related instructions |
| `voiceEnabled` | boolean | - | 启用 push-to-talk voice dictation。运行 `/voice` 时自动写入。需要 Claude.ai account |
| `showClearContextOnPlanAccept` | boolean | `false` | 在 plan accept screen 上显示 "clear context" option。设置为 `true` 恢复 option（自 v2.1.81 起默认隐藏） |
| `disableDeepLinkRegistration` | string | - | 设置为 `"disable"` 以防止 Claude Code 在启动时将 `claude-cli://` protocol handler 注册到操作系统。Deep links 让 external tools 通过 `claude-cli://open?q=...` 打开带有预填充 prompt 的 Claude Code session。在 protocol handler registration 受限或单独管理的环境中很有用 |
| `feedbackSurveyRate` | number | - | 当符合条件时 session quality survey 出现的概率（0–1）。Enterprise admins 可以控制 survey 显示的频率。示例：`0.05` = 5% 的 eligible sessions |

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

将 plan 和 auto-memory files 存储在 custom locations。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | `/plan` outputs 存储的 Directory |
| `autoMemoryDirectory` | string | - | auto-memory storage 的 Custom directory。接受 `~/`-expanded paths。不接受在 project settings（`.claude/settings.json`）中以防止将 memory writes 重定向到敏感位置；接受来自 policy, local, 和 user settings |

**Example:**
```json
{
  "plansDirectory": "./my-plans"
}
```

**Use Case:** 用于将 planning artifacts 与 Claude 的内部文件分开组织，或将 plans 保持在共享团队位置。

### Worktree Settings

配置 `--worktree` 如何创建和管理 git worktrees。用于减少大型 monorepos 中的 disk usage 和 startup time。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | 从 main repository symlink 到每个 worktree 的 Directories 以避免在 disk 上复制大型 directories |
| `worktree.sparsePaths` | array | `[]` | 通过 git sparse-checkout（cone mode）在每个 worktree 中 check out 的 Directories。只有列出的 paths 被写入 disk |

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

自定义 git commits 和 pull requests 的 attribution messages。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `attribution.commit` | string | Co-authored-by | Git commit attribution（支持 trailers） |
| `attribution.pr` | string | Generated message | Pull request description attribution |
| `includeCoAuthoredBy` | boolean | `true` | **已弃用** - 改用 `attribution` |

**Example:**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**Note:** 设置为空 string（`""`）以完全隐藏 attribution。

### Authentication Helpers

用于动态 authentication token generation 的 Scripts。

| Key | Type | Description |
|-----|------|-------------|
| `apiKeyHelper` | string | 输出 auth token 的 Shell script path（作为 `X-Api-Key` header 发送） |
| `forceLoginMethod` | string | 将 login 限制为 `"claudeai"` 或 `"console"` accounts |
| `forceLoginOrgUUID` | string | login 期间自动选择 organization 的 UUID |

**Example:**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

### Company Announcements

在启动时向用户显示自定义 announcements（随机循环）。

| Key | Type | Description |
|-----|------|-------------|
| `companyAnnouncements` | array | 启动时显示的 Strings 数组 |

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

控制 Claude 可以执行哪些 tools 和 operations。

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
| `permissions.allow` | array | 允许 tool use 而无需 prompting 的 Rules |
| `permissions.ask` | array | 需要 user confirmation 的 Rules |
| `permissions.deny` | array | 阻止 tool use 的 Rules（最高优先级） |
| `permissions.additionalDirectories` | array | Claude 可以访问的 Extra directories |
| `permissions.defaultMode` | string | Default permission mode。在 Remote environments 中，只承认 `acceptEdits` 和 `plan`（v2.1.70+） |
| `permissions.disableBypassPermissionsMode` | string | 防止 bypass mode activation |
| `allowManagedPermissionRulesOnly` | boolean | **（仅 Managed）** 仅 managed permission rules 应用；user/project `allow`, `ask`, `deny` rules 被忽略 |
| `allow_remote_sessions` | boolean | **（仅 Managed）** 允许用户启动 Remote Control 和 web sessions。默认为 `true`。设置为 `false` 以防止 remote session access *（不在 official docs 中 — official permissions page 声明 "Access to Remote Control and web sessions is not controlled by a managed settings key." 在 Team 和 Enterprise plans 上，admins 通过 [Claude Code admin settings](https://claude.ai/admin-settings/claude-code) 启用/禁用）* |
| `autoMode` | object | 自定义 [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) classifier 阻止和允许的内容。包含 `environment`（trusted infrastructure descriptions）、`allow`（block rules 的 exceptions）和 `soft_deny`（block rules） — 都是 prose strings 的 arrays。**不从 shared project settings**（`.claude/settings.json`）读取以防止 repo injection。可在 user, local, 和 managed settings 中使用。设置 `allow` 或 `soft_deny` **替换** 该部分的整个 default list。运行 `claude auto-mode defaults` 在自定义之前查看 built-in rules |
| `disableAutoMode` | string | 设置为 `"disable"` 以防止 [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) 被激活。从 `Shift+Tab` cycle 中移除 `auto` 并在启动时拒绝 `--permission-mode auto`。可在任何 settings level 设置；在 managed settings 中最有用，因为用户无法覆盖它 |
| `useAutoModeDuringPlan` | boolean | plan mode 在 auto mode 可用时是否使用 auto mode semantics。默认：`true`。不从 shared project settings（`.claude/settings.json`）读取。在 `/config` 中显示为 "Use auto mode during plan" |

### Permission Modes

| Mode | Behavior |
|------|----------|
| `"default"` | 带有 prompts 的标准 permission checking |
| `"acceptEdits"` | 自动接受 file edits 而无需询问 |
| `"askEdits"` | 在每次 operation 之前询问 *（不在 official docs 中 — 未验证）* |
| `"dontAsk"` | 自动拒绝 tools 除非通过 `/permissions` 或 `permissions.allow` rules 预先批准 |
| `"viewOnly"` | Read-only mode，无 modifications *（不在 official docs 中 — 未验证）* |
| `"bypassPermissions"` | 跳过所有 permission checks（危险） |
| `"auto"` | Background classifier 替换 manual prompts。Research preview — 需要 Team plan + Sonnet/Opus 4.6。Classifier 自动批准 read-only 和 file edits；将其他所有内容通过 safety check。在连续 3 次或总共 20 次 blocks 后回退到 prompting。使用 `autoMode` setting 配置 |
| `"plan"` | Read-only exploration mode |

### Tool Permission Syntax

| Tool | Syntax | Examples |
|------|--------|----------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`, `Bash(* install)`, `Bash(git * main)` |
| `Read` | `Read(path pattern)` | `Read(.env)`, `Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`, `Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`, `Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | Global web search |
| `Task` | `Task(agent-name)` | `Task(Explore)`, `Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`, `Agent(*)` — permission scoped to subagent spawning |
| `Skill` | `Skill(skill-name)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` 或 `MCP(server:tool)` | `mcp__memory__*`, `MCP(github:*)` |

**Evaluation order:** Rules 按顺序评估：deny rules 优先，然后 ask，然后 allow。第一个匹配的 rule 获胜。

**Read/Edit path patterns:** `Read`, `Edit`, 和 `Write` 的 Permission rules 支持 gitignore-style patterns 并有四种 prefix types：

| Prefix | Meaning | Example |
|--------|---------|---------|
| `//` | 来自 filesystem root 的 Absolute path | `Read(//Users/alice/file)` |
| `~/` | 相对于 home directory | `Read(~/.zshrc)` |
| `/` | 相对于 project root | `Edit(/src/**)` |
| `./` 或 none | Relative path（current directory） | `Read(.env)`, `Read(*.ts)` |

**Bash wildcard notes:**
- `*` 可以出现在**任何位置**：prefix（`Bash(* install)`）、suffix（`Bash(npm *)`）或 middle（`Bash(git * main)`）
- **Word boundary:** `Bash(ls *)`（`*` 前有空格）匹配 `ls -la` 但不匹配 `lsof`；`Bash(ls*)`（无空格）匹配两者
- `Bash(*)` 被视为等同于 `Bash`（匹配所有 bash commands）
- Permission rules 支持 output redirections：`Bash(python:*)` 匹配 `python script.py > output.txt`
- 旧版 `:*` suffix syntax（例如 `Bash(npm:*)`）等同于 ` *` 但已弃用

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

Hook configuration（events, properties, matchers, exit codes, environment variables, 和 HTTP hooks）在专用仓库中维护：

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — 完整的 hook reference，包含 sound notification system、所有 19 hook events、HTTP hooks、matcher patterns、exit codes 和 environment variables。

Hook-related settings keys（`hooks`, `disableAllHooks`, `allowManagedHooksOnly`, `allowedHttpHookUrls`, `httpHookAllowedEnvVars`）在那里有文档。

对于 official hooks reference，参见 [Claude Code Hooks Documentation](https://code.claude.com/docs/en/hooks)。

---

## MCP Servers

配置 Model Context Protocol servers 以扩展功能。

### MCP Settings

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enableAllProjectMcpServers` | boolean | Any | 自动批准所有 `.mcp.json` servers |
| `enabledMcpjsonServers` | array | Any | Allowlist specific server names |
| `disabledMcpjsonServers` | array | Any | Blocklist specific server names |
| `allowedMcpServers` | array | Managed only | Allowlist 带有 name/command/URL matching |
| `deniedMcpServers` | array | Managed only | Blocklist 带有 matching |
| `allowManagedMcpServersOnly` | boolean | Managed only | 仅允许 managed allowlist 中明确列出的 MCP servers |
| `channelsEnabled` | boolean | Managed only | 允许 [channels](https://code.claude.com/docs/en/channels) 用于 Team 和 Enterprise users。未设置或 `false` 时，无论 `--channels` flag 如何都阻止 channel message delivery |
| `allowedChannelPlugins` | array | Managed only | 允许推送消息的 channel plugins 的 Allowlist。设置时替换默认 Anthropic allowlist。Undefined = 回退到默认，empty array = 阻止所有 channel plugins。需要 `channelsEnabled: true`。每个 entry 是带有 `marketplace` 和 `plugin` fields 的 object（v2.1.84） |

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

为安全性配置 bash command sandboxing。

### Sandbox Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `sandbox.enabled` | boolean | `false` | 启用 bash sandboxing |
| `sandbox.failIfUnavailable` | boolean | `false` | 当启用 sandbox 但无法启动时以 error 退出，而非 unsandboxed 运行。用于需要严格 sandboxing 的 enterprise policies（v2.1.83） |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | Sandboxed 时自动批准 bash |
| `sandbox.excludedCommands` | array | `[]` | 在 sandbox 外运行的 Commands |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | 允许 `dangerouslyDisableSandbox` |
| `sandbox.ignoreViolations` | object | `{}` | Command patterns 到 path arrays 的 Map — 抑制 violation warnings *（在 JSON schema 中，不在 official settings page 上）* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | Docker 的较弱 sandbox（降低安全性） |
| `sandbox.network.allowUnixSockets` | array | `[]` | sandbox 中可访问的 Specific Unix socket paths |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | 允许所有 Unix sockets（覆盖 allowUnixSockets） |
| `sandbox.network.allowLocalBinding` | boolean | `false` | 允许绑定到 localhost ports（macOS） |
| `sandbox.network.allowedDomains` | array | `[]` | sandbox 的 Network domain allowlist |
| `sandbox.network.deniedDomains` | array | `[]` | sandbox 的 Network domain denylist *（不在 official docs 中 — 未验证）* |
| `sandbox.network.httpProxyPort` | number | - | HTTP proxy port 1-65535（custom proxy） |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 proxy port 1-65535（custom proxy） |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | 仅允许 managed allowlist 中的 domains（managed settings） |
| `sandbox.filesystem.allowWrite` | array | `[]` | sandboxed commands 可以写入的 Additional paths。Arrays 在所有 settings scopes 中合并。Prefix: `/`（absolute）、`~/`（home）、`./` 或 none（project settings 中为 project-relative，user settings 中为 `~/.claude`-relative）。旧版 `//` prefix 用于 absolute paths 仍然有效。**Note:** 这与 [Read/Edit permission rules](#tool-permission-syntax) 不同，后者使用 `//` 表示 absolute，`/` 表示 project-relative |
| `sandbox.filesystem.denyWrite` | array | `[]` | sandboxed commands 不能写入的 Paths。Arrays 在所有 settings scopes 中合并。与 `allowWrite` 相同的路径 prefix conventions |
| `sandbox.filesystem.denyRead` | array | `[]` | sandboxed commands 不能读取的 Paths。Arrays 在所有 settings scopes 中合并。与 `allowWrite` 相同的路径 prefix conventions |
| `sandbox.filesystem.allowRead` | array | `[]` | 在 `denyRead` regions 内重新允许读取访问的 Paths。优先级高于 `denyRead`。Arrays 在所有 settings scopes 中合并。与 `allowWrite` 相同的路径 prefix conventions |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **（仅 Managed）** 仅尊重来自 managed settings 的 `allowRead` paths。来自 user, project, 和 local settings 的 `allowRead` entries 被忽略 |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | （仅 macOS）允许访问 system TLS trust（`com.apple.trustd.agent`）；降低安全性 |

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

配置 Claude Code plugins 和 marketplaces。

### Plugin Settings

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enabledPlugins` | object | Any | 启用/禁用 specific plugins |
| `extraKnownMarketplaces` | object | Project | 添加 custom plugin marketplaces（通过 `.claude/settings.json` 团队共享） |
| `strictKnownMarketplaces` | array | Managed only | 允许的市场places 的 Allowlist |
| `skippedMarketplaces` | array | Any | 用户拒绝安装的市场places *（在 JSON schema 中，不在 official settings page 上）* |
| `skippedPlugins` | array | Any | 用户拒绝安装的 Plugins *（在 JSON schema 中，不在 official settings page 上）* |
| `pluginConfigs` | object | Any | Per-plugin MCP server configs（以 `plugin@marketplace` 为 key） *（在 JSON schema 中，不在 official settings page 上）* |
| `blockedMarketplaces` | array | Managed only | 阻止 specific plugin marketplaces |
| `pluginTrustMessage` | string | Managed only | 提示用户信任 plugins 时显示的 Custom message |

**Marketplace source types:** `github`, `git`, `directory`, `hostPattern`, `settings`, `url`, `npm`, `file`。使用 `source: 'settings'` 来 inline 声明少量 plugins 而无需设置 hosted marketplace repository。

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
| `"default"` | 推荐用于你的 account type |
| `"sonnet"` | Latest Sonnet model（Claude Sonnet 4.6） |
| `"opus"` | Latest Opus model（Claude Opus 4.6） |
| `"haiku"` | Fast Haiku model |
| `"sonnet[1m]"` | Sonnet with 1M token context |
| `"opus[1m]"` | Opus with 1M token context（自 v2.1.75 起在 Max, Team, 和 Enterprise 上默认） |
| `"opusplan"` | Opus for planning, Sonnet for execution |

**Example:**
```json
{
  "model": "opus"
}
```

### Model Overrides

将 Anthropic model IDs 映射到 provider-specific model IDs 用于 Bedrock, Vertex, 或 Foundry deployments。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `effortLevel` | string | - | 在 sessions 之间持久化 effort level。接受 `"low"`, `"medium"`, 或 `"high"`。运行 `/effort low`, `/effort medium`, 或 `/effort high` 时自动写入。支持 Opus 4.6 和 Sonnet 4.6 |
| `modelOverrides` | object | - | 将 model picker entries 映射到 provider-specific IDs（例如 Bedrock inference profile ARNs）。每个 key 是 model picker entry name，每个 value 是 provider model ID |

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

`/model` command 公开一个 **effort level** 控制，调整模型每次响应应用的 reasoning 量。在 `/model` UI 中使用 ← → arrow keys 循环遍历 effort levels。

| Effort Level | Description |
|-------------|-------------|
| High | Full reasoning depth，最适合 complex tasks |
| Medium (default) | Balanced reasoning，适合 everyday tasks |
| Low | Minimal reasoning，最快响应 |

**How to use:**
1. 运行 `/effort low`, `/effort medium`, 或 `/effort high` 直接设置（v2.1.76+）
2. 或运行 `/model` → select a model → 使用 **← →** arrow keys 调整
3. 设置通过 `settings.json` 中的 `effortLevel` key 持久化

**Note:** Effort level 在 Max 和 Team plans 上可用于 Opus 4.6 和 Sonnet 4.6。默认值在 v2.1.68 中从 High 改为 Medium。截至 v2.1.75，Opus 4.6 的 1M context window 在 Max, Team, 和 Enterprise plans 上默认可用。

### Model Environment Variables

通过 `env` key 配置：

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
| `outputStyle` | string | `"default"` | Output style（例如 `"Explanatory"`） |
| `spinnerTipsEnabled` | boolean | `true` | 等待时显示 tips |
| `spinnerVerbs` | object | - | Custom spinner verbs 带有 `mode`（"append" 或 "replace"）和 `verbs` array |
| `spinnerTipsOverride` | object | - | Custom spinner tips 带有 `tips`（string array）和可选的 `excludeDefault`（boolean） |
| `respectGitignore` | boolean | `true` | 在 file picker 中 Respect .gitignore |
| `prefersReducedMotion` | boolean | `false` | 减少 UI 中的 animations 和 motion effects |
| `fileSuggestion` | object | - | Custom file suggestion command（参见下面的 File Suggestion Configuration） |

### Global Config Settings (`~/.claude.json`)

这些 display preferences 存储在 `~/.claude.json` 中，**不是** `settings.json`。将它们添加到 `settings.json` 会触发 schema validation error。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `autoConnectIde` | boolean | `false` | 当 Claude Code 从 external terminal 启动时自动连接到 running IDE。在 VS Code 或 JetBrains terminal 外运行时在 `/config` 中显示为 **Auto-connect to IDE (external terminal)** |
| `autoInstallIdeExtension` | boolean | `true` | 从 VS Code terminal 运行时自动安装 Claude Code IDE extension。在 `/config` 中显示为 **Auto-install IDE extension**。也可通过 `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` env var 禁用 |
| `editorMode` | string | `"normal"` | input prompt 的 Key binding mode：`"normal"` 或 `"vim"`。运行 `/vim` 时自动写入。在 `/config` 中显示为 **Key binding mode** |
| `showTurnDuration` | boolean | `true` | 在 responses 后显示 turn duration messages（例如 "Cooked for 1m 6s"）。直接编辑 `~/.claude.json` 更改 |
| `terminalProgressBarEnabled` | boolean | `true` | 在支持的 terminals 中显示 terminal progress bar（ConEmu, Ghostty 1.2.0+, 和 iTerm2 3.6.6+）。在 `/config` 中显示为 **Terminal progress bar** |
| `teammateMode` | string | `"auto"` | [agent team](https://code.claude.com/docs/en/agent-teams) teammates 如何显示：`"auto"`（在 tmux 或 iTerm2 中选择 split panes，否则 in-process）、`"in-process"` 或 `"tmux"`。参见 [choose a display mode](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) |

### Status Line Configuration

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

**Status Line Input Fields:**

status line command 在 stdin 上接收带有以下 notable fields 的 JSON object：

| Field | Description |
|-------|-------------|
| `workspace.added_dirs` | 通过 `/add-dir` 添加的 Directories |
| `context_window.used_percentage` | Context window usage percentage |
| `context_window.remaining_percentage` | Context window remaining percentage |
| `current_usage` | Current context window token count |
| `exceeds_200k_tokens` | context 是否超过 200k tokens |
| `rate_limits.five_hour.used_percentage` | Five-hour rate limit usage percentage（v2.1.80+） |
| `rate_limits.five_hour.resets_at` | Five-hour rate limit reset timestamp |
| `rate_limits.seven_day.used_percentage` | Seven-day rate limit usage percentage |
| `rate_limits.seven_day.resets_at` | Seven-day rate limit reset timestamp |

### File Suggestion Configuration

file suggestion script 在 stdin 上接收 JSON object（例如 `{"query": "src/comp"}`）并必须输出最多 15 个 file paths（每行一个）。

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
| `awsAuthRefresh` | string | 刷新 AWS auth 的 Script（修改 `.aws` dir） |
| `awsCredentialExport` | string | 输出带有 AWS credentials 的 JSON 的 Script |

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
| `otelHeadersHelper` | string | 生成动态 OpenTelemetry headers 的 Script |

**Example:**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## Environment Variables (via `env`)

为所有 Claude Code sessions 设置 environment variables。

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

（此处省略详细列表，与源文件相同）

---

## Useful Commands

| Command | Description |
|---------|-------------|
| `/model` | Switch models 并调整 Opus 4.6 effort level |
| `/effort` | 直接设置 effort level：`low`, `medium`, `high`（v2.1.76+） |
| `/config` | Interactive configuration UI |
| `/memory` | View/edit all memory files |
| `/agents` | Manage subagents |
| `/mcp` | Manage MCP servers |
| `/hooks` | View configured hooks |
| `/plugin` | Manage plugins |
| `/keybindings` | Configure custom keyboard shortcuts |
| `/skills` | View and manage skills |
| `/permissions` | View and manage permission rules |
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
  "includeGitInstructions": true,
  "defaultShell": "bash",
  "plansDirectory": "./plans",
  "effortLevel": "medium",

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ]
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

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium"
  }
}
```

---

## Sources

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code Settings JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub Settings Examples](https://github.com/feiskyer/claude-code-settings)
- [Eesel AI - Developer's Guide to settings.json](https://www.eesel.ai/blog/settings-json-claude-code)
- [Shipyard - Claude Code CLI Cheatsheet](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Claude Code Environment Variables Reference](https://code.claude.com/docs/en/env-vars)
- [Claude Code Permissions Reference](https://code.claude.com/docs/en/permissions)

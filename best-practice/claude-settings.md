# Settings 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2026%2C%202026%201%3A10%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.119-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../.claude/settings.json)

Claude Code `settings.json` 文件中所有可用配置选项的综合指南。截至 v2.1.119，Claude Code 提供了 **60+ 设置项** 和 **175+ 环境变量**（使用 `settings.json` 中的 `"env"` 字段来避免使用包装脚本）。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 目录

1. [设置层级](#settings-hierarchy)
2. [核心配置](#core-configuration)
3. [权限](#permissions)
4. [Hooks](#hooks)
5. [MCP 服务器](#mcp-servers)
6. [沙箱](#sandbox)
7. [插件](#plugins)
8. [模型配置](#model-configuration)
9. [显示与 UX](#display--ux)
10. [AWS 与云凭据](#aws--cloud-credentials)
11. [环境变量（通过 env 设置）](#environment-variables-via-env)
12. [常用命令](#useful-commands)

---

## 设置层级

设置按优先级顺序应用（从高到低）：

| 优先级 | 位置 | 范围 | 共享？ | 用途 |
|----------|----------|-------|---------|---------|
| 1 | 托管设置 | 组织 | 是（由 IT 部署） | 无法被覆盖的安全策略 |
| 2 | 命令行参数 | 会话 | N/A | 临时单次会话覆盖 |
| 3 | `.claude/settings.local.json` | 项目 | 否（被 git 忽略） | 个人项目级 |
| 4 | `.claude/settings.json` | 项目 | 是（已提交） | 团队共享设置 |
| 5 | `~/.claude/settings.json` | 用户 | N/A | 全局个人默认 |

**托管设置**由组织强制执行，无法被任何其他级别（包括命令行参数）覆盖。传递方式：
- **服务器托管**设置（远程传递）
- **MDM 配置文件** —— macOS plist 位于 `com.anthropic.claudecode`
- **注册表策略** —— Windows `HKLM\SOFTWARE\Policies\ClaudeCode`（管理员）和 `HKCU\SOFTWARE\Policies\ClaudeCode`（用户级，最低策略优先级）
- **文件** —— `managed-settings.json` 和 `managed-mcp.json`（macOS：`/Library/Application Support/ClaudeCode/`，Linux/WSL：`/etc/claude-code/`，Windows：`C:\Program Files\ClaudeCode\`）
- **Drop-in 目录** —— `managed-settings.d/` 与 `managed-settings.json` 并列，用于独立的策略片段（v2.1.83）。遵循 systemd 惯例，`managed-settings.json` 首先作为基础合并，然后按字母顺序排序并合并 drop-in 目录中的所有 `*.json` 文件。后面的文件覆盖前面的标量值；数组被连接和去重；对象被深度合并。以 `.` 开头的隐藏文件被忽略。使用数字前缀控制合并顺序（如 `10-telemetry.json`、`20-security.json`）

在托管层级内，优先级为：服务器托管 > MDM/OS 级策略 > 基于文件（`managed-settings.d/*.json` + `managed-settings.json`）> HKCU 注册表（仅 Windows）。只使用一种托管来源；来源之间不跨层级合并。在基于文件的层级内，drop-in 文件和基础文件会被合并。

> **注意：** 截至 v2.1.75，已弃用的 Windows 回退路径 `C:\ProgramData\ClaudeCode\managed-settings.json` 已被移除。请使用 `C:\Program Files\ClaudeCode\managed-settings.json`。

**重要：**
- `deny` 规则具有最高安全优先级，无法被较低优先级的 allow/ask 规则覆盖。
- 托管设置可能会锁定或覆盖本地行为，即使本地文件指定了不同的值。
- 数组设置（如 `permissions.allow`）在各层级之间**连接并去重** —— 来自所有层级的条目被合并，而不是替换。

---

## 核心配置

### 通用设置

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `$schema` | string | - | 用于 IDE 验证和自动补全的 JSON Schema URL（如 `"https://json.schemastore.org/claude-code-settings.json"`） |
| `model` | string | `"default"` | 覆盖默认模型。接受别名（`sonnet`、`opus`、`haiku`）或完整模型 ID |
| `agent` | string | - | 设置主对话的默认 Agent。值是 `.claude/agents/` 中的 Agent 名称。也可通过 `--agent` CLI 标志设置 |
| `language` | string | `"english"` | Claude 的偏好回复语言。同时设置语音听写语言 |
| `cleanupPeriodDays` | number | `30` | 启动清理扫描的期限截止天数（最小为 1）。不活跃的会话记录和孤立 subagent worktree 将被删除；截至 v2.1.117，扫描还涵盖 `~/.claude/tasks/`、`~/.claude/shell-snapshots/` 和 `~/.claude/backups/`。设置为 `0` 会被验证错误拒绝。要在非交互模式（`-p`）下禁用记录写入，请使用 `--no-session-persistence` 或 SDK 选项 `persistSession: false` |
| `autoUpdatesChannel` | string | `"latest"` | 发布频道：`"stable"` 或 `"latest"` |
| `minimumVersion` | string | - | 防止自动更新降级到特定版本以下。在切换到稳定频道并选择停留在当前版本直到稳定版追平时自动设置。与 `autoUpdatesChannel` 配合使用 |
| `alwaysThinkingEnabled` | boolean | `false` | 为所有会话默认启用扩展思考 |
| `skipWebFetchPreflight` | boolean | `false` | 在获取 URL 之前跳过 WebFetch 阻止列表检查（在 JSON schema 中，不在官方设置页面） |
| `availableModels` | array | - | 限制用户通过 `/model`、`--model`、Config 工具或 `ANTHROPIC_MODEL` 可选择的模型。不影响 Default 选项。示例：`["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | 要求用户每次会话都选择加入快速模式 |
| `defaultShell` | string | `"bash"` | 输入框 `!` 命令的默认 shell。接受 `"bash"`（默认）或 `"powershell"`。设置为 `"powershell"` 会在 Windows 上将交互式 `!` 命令路由到 PowerShell。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`（v2.1.84） |
| `includeGitInstructions` | boolean | `true` | 在 Claude 的系统提示中包含内置的 commit 和 PR 工作流指令以及 git 状态快照。设置 `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` 环境变量优先于此设置 |
| `voice` | object | - | 语音听写配置。包含三个字段的对象：`enabled`（布尔值 —— 按键说话的开关）、`mode`（字符串 —— `"hold"` 为按住说话或 `"tap"` 为点击切换）和 `autoSubmit`（布尔值 —— 听写结束时立即提交）。运行 `/voice` 时自动写入。需要 Claude.ai 账户（v2.1.118 扩展结构） |
| `voiceEnabled` | boolean | - | **已弃用** —— `voice.enabled` 的旧版别名。请使用 `voice` 对象以获取 `mode` 和 `autoSubmit` 控制 |
| `showClearContextOnPlanAccept` | boolean | `false` | 在 Plan 接受屏幕上显示"清除 Context"选项。设置为 `true` 以恢复该选项（自 v2.1.81 起默认隐藏） |
| `viewMode` | string | - | 启动时的默认记录视图模式：`"default"`、`"verbose"` 或 `"focus"`。设置时覆盖粘性 Ctrl+O 选择 |
| `disableDeepLinkRegistration` | string | - | 设置为 `"disable"` 以阻止 Claude Code 在启动时将 `claude-cli://` 协议处理器注册到操作系统。深度链接让外部工具可以通过 `claude-cli://open?q=...` 打开带有预填 prompt 的 Claude Code 会话。`q` 参数支持使用 URL 编码换行符（`%0A`）的多行 prompt。在协议处理器注册受限或单独管理的环境中有用 |
| `showThinkingSummaries` | boolean | `false` | 在交互会话中显示扩展思考摘要。当未设置或为 `false`（交互模式默认）时，思考块被 API 编辑并以折叠存根显示。编辑只改变你看到的内容，不改变模型生成的内容 —— 要减少思考花费，请降低预算或禁用思考。非交互模式（`-p`）和 SDK 调用者无论此设置如何都始终接收摘要 |
| `disableSkillShellExecution` | boolean | `false` | 禁用来自用户、项目、插件或附加目录的 Skill 和自定义 Command 中 `` !`...` `` 和 `` ```! `` 块的内联 shell 执行。命令被替换为 `[shell command execution disabled by policy]` 而非被执行。捆绑和管理型 Skill 不受影响（v2.1.91） |
| `forceRemoteSettingsRefresh` | boolean | `false` | **（仅托管）** 阻止 CLI 启动直到远程托管设置被新鲜获取。如果获取失败，CLI 退出（失败关闭）。用于企业环境中策略执行必须在任何会话开始前保持最新（v2.1.92） |
| `wslInheritsWindowsSettings` | boolean | `false` | **（仅 Windows 托管设置）** 当设置为 `true` 时，WSL 上的 Claude Code 从 Windows 策略链（HKLM 注册表 + `C:\Program Files\ClaudeCode\managed-settings.json`）以及 `/etc/claude-code` 读取托管设置，Windows 来源优先。仅在 HKLM 注册表键或 `C:\Program Files\ClaudeCode\managed-settings.json` 中设置时才生效，两者都需要 Windows 管理员权限写入。要让 HKCU 策略也在 WSL 上生效，还必须在 HKCU 本身中设置该标志。对原生 Windows 无效（v2.1.118） |
| `tui` | string | `"default"` | 渲染模式：`"fullscreen"` 或 `"default"`。通过 `/tui fullscreen` 设置为无闪烁的 alt-screen 渲染（v2.1.110） |
| `awaySummaryEnabled` | boolean | `true` | 当用户离开后返回时生成"离开摘要"（闲置会话回顾）。设置为 `false` 以退出。与 `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` 环境变量配对（v2.1.110） |
| `feedbackSurveyRate` | number | - | 会话质量调查在符合条件时出现的概率（0-1）。企业管理员可以控制调查显示的频率。示例：`0.05` = 5% 符合条件的会话 |

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

### 计划与记忆目录

将计划和自动记忆文件存储在自定义位置。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | `/plan` 输出存储的目录 |
| `autoMemoryDirectory` | string | - | 自动记忆存储的自定义目录。接受 `~/` 展开的路径。不接受在项目设置（`.claude/settings.json`）中，以防止将记忆写入重定向到敏感位置；可从策略、本地和用户设置中接受 |

**示例：**
```json
{
  "plansDirectory": "./my-plans"
}
```

**用途：** 有助于将规划产物与 Claude 的内部文件分开整理，或将计划保存在共享的团队位置。

### Worktree 设置

配置 `--worktree` 如何创建和管理 git worktree。有助于在大型 monorepo 中减少磁盘使用和启动时间。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | 要从主仓库符号链接到每个 worktree 的目录，以避免在磁盘上复制大型目录 |
| `worktree.sparsePaths` | array | `[]` | 通过 git sparse-checkout（cone 模式）在每个 worktree 中签出的目录。只有列出的路径会被写入磁盘 |

### 归属设置

自定义 git commit 和 pull request 的归属消息。

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `attribution.commit` | string | Co-authored-by | Git commit 归属（支持 trailers） |
| `attribution.pr` | string | 生成的消息 | Pull request 描述归属 |
| `prUrlTemplate` | string | - | 控制 commit 归属中"PR"徽章如何链接到 PR UI 的 URL 模板。支持仓库主机、所有者、仓库和 PR 编号的占位符。适用于自托管的 GitLab/Bitbucket/GitHub Enterprise 实例，默认的 `https://github.com/...` URL 不适用（v2.1.119） |
| `includeCoAuthoredBy` | boolean | `true` | **已弃用** —— 使用 `attribution` 代替 |

**示例：**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**注意：** 设置为空字符串（`""`）以完全隐藏归属。

### 认证辅助

用于动态认证令牌生成的脚本。

| Key | Type | Description |
|-----|------|-------------|
| `apiKeyHelper` | string | 输出认证令牌的脚本路径（作为 `X-Api-Key` 头发送） |
| `forceLoginMethod` | string | 限制登录为 `"claudeai"` 或 `"console"` 账户 |
| `forceLoginOrgUUID` | string \| array | 要求登录属于特定组织。接受单个 UUID 字符串（在登录时也预选该组织）或 UUID 数组，其中任何列出的组织都被接受且不预选。在托管设置中设置时，如果认证账户不属于列出的组织则登录失败；空数组会导致失败关闭并以配置错误消息阻止登录 |

### 公司公告

在启动时向用户显示自定义公告（随机循环）。

| Key | Type | Description |
|-----|------|-------------|
| `companyAnnouncements` | array | 启动时显示的字符串数组 |

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

## 权限

控制 Claude 可以使用的工具和操作。

### 权限结构

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

### 权限键

| Key | Type | Description |
|-----|------|-------------|
| `permissions.allow` | array | 允许使用工具而无需提示的规则 |
| `permissions.ask` | array | 需要用户确认的规则 |
| `permissions.deny` | array | 阻止使用工具的规则（最高优先级） |
| `permissions.additionalDirectories` | array | Claude 可以访问的额外目录 |
| `permissions.defaultMode` | string | 默认权限模式。在远程环境中，仅尊重 `acceptEdits` 和 `plan`（v2.1.70+） |
| `permissions.disableBypassPermissionsMode` | string | 阻止绕过模式激活 |
| `permissions.skipDangerousModePermissionPrompt` | boolean | 跳过通过 `--dangerously-skip-permissions` 或 `defaultMode: "bypassPermissions"` 进入绕过权限模式时显示的确认提示。在项目设置（`.claude/settings.json`）中设置时被忽略，以防止不可信的仓库自动绕过提示 |
| `allowManagedPermissionRulesOnly` | boolean | **（仅托管）** 仅应用托管权限规则；用户/项目的 `allow`、`ask`、`deny` 规则被忽略 |
| `autoMode` | object | 自定义 [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) 分类器阻止和允许的内容。包含 `environment`（受信任的基础设施描述）、`allow`（阻止规则的例外）和 `soft_deny`（阻止规则）—— 均为散文字符串数组。**不从共享项目设置**（`.claude/settings.json`）中读取，以防止仓库注入。可在用户、本地和托管设置中使用。设置 `allow` 或 `soft_deny` **会替换**整个默认列表，除非你在数组中包含字面量字符串 `"$defaults"` —— 该哨兵在此位置继承内置规则，以便自定义条目与其并列添加（v2.1.118）。在自定义之前运行 `claude auto-mode defaults` 查看内置规则 |
| `disableAutoMode` | string | 设置为 `"disable"` 以阻止 [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) 被激活。从 `Shift+Tab` 循环中移除 `auto` 并在启动时拒绝 `--permission-mode auto`。可在任何设置级别设置；在用户无法覆盖的托管设置中最有用 |
| `useAutoModeDuringPlan` | boolean | Plan Mode 在 auto mode 可用时是否使用 auto mode 语义。默认：`true`。不从共享项目设置（`.claude/settings.json`）中读取。在 `/config` 中显示为"在 plan 期间使用 auto mode" |

### 权限模式

| 模式 | 行为 |
|------|----------|
| `"default"` | 标准权限检查并提示 |
| `"acceptEdits"` | 自动接受文件编辑而无需询问 |
| `"dontAsk"` | 自动拒绝工具，除非通过 `/permissions` 或 `permissions.allow` 规则预先批准 |
| `"bypassPermissions"` | 跳过所有权限检查（危险） |
| `"auto"` | 使用后台安全检查自动批准工具调用，验证操作是否与你的请求一致。研究预览。分类器自动批准只读和文件编辑；将其他所有内容通过安全检查。在连续 3 次或总共 20 次阻止后回退到提示。自 v2.1.111 起为默认 `Shift+Tab` 权限模式循环的一部分（`--enable-auto-mode` 标志在 v2.1.111 中移除 —— 使用 `--permission-mode auto` 以此模式启动）。使用 `autoMode` 设置进行配置 |
| `"plan"` | 只读探索模式 |

### 工具权限语法

| 工具 | 语法 | 示例 |
|------|--------|----------|
| `Bash` | `Bash(命令模式)` | `Bash(npm run *)`、`Bash(* install)`、`Bash(git * main)` |
| `Read` | `Read(路径模式)` | `Read(.env)`、`Read(./secrets/**)` |
| `Edit` | `Edit(路径模式)` | `Edit(src/**)`、`Edit(*.ts)` |
| `Write` | `Write(路径模式)` | `Write(*.md)`、`Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(模式)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:模式)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | 全局网页搜索 |
| `Task` | `Task(agent 名称)` | `Task(Explore)`、`Task(my-agent)` |
| `Agent` | `Agent(名称)` | `Agent(researcher)`、`Agent(*)` —— 权限限定于 subagent 生成 |
| `Skill` | `Skill(skill 名称)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` 或 `MCP(server:tool)` | `mcp__memory__*`、`MCP(github:*)` |

**评估顺序：** 规则按顺序评估：先 deny 规则，然后 ask，最后 allow。第一个匹配的规则生效。

**Read/Edit 路径模式：** `Read`、`Edit` 和 `Write` 的权限规则支持 gitignore 风格的模式，有四种前缀类型：

| 前缀 | 含义 | 示例 |
|--------|---------|---------|
| `//` | 从文件系统根目录的绝对路径 | `Read(//Users/alice/file)` |
| `~/` | 相对于主目录 | `Read(~/.zshrc)` |
| `/` | 相对于项目根目录 | `Edit(/src/**)` |
| `./` 或无 | 相对路径（当前目录） | `Read(.env)`、`Read(*.ts)` |

**Bash 通配符说明：**
- `*` 可以出现在**任何位置**：前缀（`Bash(* install)`）、后缀（`Bash(npm *)`）或中间（`Bash(git * main)`）
- **词边界：** `Bash(ls *)`（`*` 前有空格）匹配 `ls -la` 但不匹配 `lsof`；`Bash(ls*)`（无空格）两者都匹配
- `Bash(*)` 被视为等同于 `Bash`（匹配所有 bash 命令）
- 权限规则支持输出重定向：`Bash(python:*)` 匹配 `python script.py > output.txt`
- 旧版 `:*` 后缀语法（如 `Bash(npm:*)`）等同于 ` *` 但已弃用

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

Hook 配置（事件、属性、匹配器、退出码、环境变量和 HTTP hooks）维护在专用仓库中：

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** —— 完整的 Hook 参考，包含声音通知系统、所有 25 个 hook 事件、HTTP hooks、匹配器模式、退出码和环境变量。

Hook 相关设置键（`hooks`、`disableAllHooks`（也禁用任何自定义状态栏）、`allowManagedHooksOnly`、`allowedHttpHookUrls`、`httpHookAllowedEnvVars`）在该仓库中记录。

有关官方 hooks 参考，请参阅 [Claude Code Hooks 文档](https://code.claude.com/docs/en/hooks)。

---

## MCP 服务器

配置 Model Context Protocol 服务器以扩展功能。

> **OAuth（v2.1.111）：** 通过 OAuth 认证的 MCP 服务器遵循 [RFC 9728](https://datatracker.ietf.org/doc/rfc9728/) 进行受保护资源元数据发现。兼容的服务器在 `/.well-known/oauth-protected-resource` 下公开授权端点，Claude Code 自动完成 OAuth 流程 —— 对于符合规范的服务器无需手动 `apiKeyHelper` 或 `headersHelper` 脚本。

### MCP 设置

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enableAllProjectMcpServers` | boolean | 任意 | 自动批准所有 `.mcp.json` 服务器 |
| `enabledMcpjsonServers` | array | 任意 | 允许列表特定服务器名称 |
| `disabledMcpjsonServers` | array | 任意 | 阻止列表特定服务器名称 |
| `allowedMcpServers` | array | 仅托管 | 允许列表，支持名称/命令/URL 匹配 |
| `deniedMcpServers` | array | 仅托管 | 阻止列表，支持匹配 |
| `allowManagedMcpServersOnly` | boolean | 仅托管 | 仅允许托管允许列表中明确列出的 MCP 服务器 |
| `channelsEnabled` | boolean | 仅托管 | 允许 Team 和 Enterprise 用户使用 [channels](https://code.claude.com/docs/en/channels)。当未设置或为 `false` 时，无论 `--channels` 标志如何，channel 消息传递都会被阻止 |
| `allowedChannelPlugins` | array | 仅托管 | 允许推送消息的 channel 插件允许列表。设置时替换默认的 Anthropic 允许列表。未定义 = 回退到默认值，空数组 = 阻止所有 channel 插件。需要 `channelsEnabled: true`。每个条目是包含 `marketplace` 和 `plugin` 字段的对象（v2.1.84） |

### MCP 服务器匹配（托管设置）

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

---

## 沙箱

配置 bash 命令沙箱以增强安全性。

### 沙箱设置

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `sandbox.enabled` | boolean | `false` | 启用 bash 沙箱 |
| `sandbox.failIfUnavailable` | boolean | `false` | 当沙箱启用但无法启动时退出并报错，而非在无沙箱下运行。适用于需要严格沙箱的企业策略（v2.1.83） |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | 沙箱化时自动批准 bash |
| `sandbox.excludedCommands` | array | `[]` | 在沙箱外运行的命令 |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | 允许 `dangerouslyDisableSandbox`。设置为 `false` 时，完全禁用逃生舱，所有命令必须在沙箱中运行（或在 `excludedCommands` 中）。适用于需要严格沙箱的企业策略 |
| `sandbox.ignoreViolations` | object | `{}` | 命令模式到路径数组的映射 —— 抑制违规警告（在 JSON schema 中，不在官方设置页面） |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | **（仅 Linux 和 WSL2）** 在无特权 Docker 环境中启用较弱的沙箱（降低安全性） |
| `sandbox.network.allowUnixSockets` | array | `[]` | **（仅 macOS）** 沙箱中可访问的特定 Unix socket 路径。在 Linux 和 WSL2 上被忽略，因为 seccomp 过滤器无法检查 socket 路径；请改用 `allowAllUnixSockets` |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | 允许所有 Unix socket（覆盖 `allowUnixSockets`）。在 Linux 和 WSL2 上，这是允许 Unix socket 的唯一方式，因为它跳过了原本会阻止 `socket(AF_UNIX, ...)` 调用的 seccomp 过滤器 |
| `sandbox.network.allowLocalBinding` | boolean | `false` | 允许绑定到 localhost 端口（macOS） |
| `sandbox.network.allowedDomains` | array | `[]` | 沙箱的网络域名允许列表 |
| `sandbox.network.deniedDomains` | array | `[]` | bash 沙箱的网络域名阻止列表。优先级高于 `allowedDomains` 中的通配符。支持 glob 模式（如 `"*.example.com"`）（v2.1.113） |
| `sandbox.network.httpProxyPort` | number | - | HTTP 代理端口 1-65535（自定义代理） |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 代理端口 1-65535（自定义代理） |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | 仅允许托管允许列表中的域名（托管设置） |
| `sandbox.network.allowMachLookup` | array | `[]` | （仅 macOS）沙箱可查找的额外 XPC/Mach 服务名称。支持单个尾部 `*` 进行前缀匹配。需要通过 XPC 通信的工具（如 iOS 模拟器或 Playwright）所需。示例：`["com.apple.coresimulator.*"]` |
| `sandbox.filesystem.allowWrite` | array | `[]` | 沙箱命令可以写入的额外路径。数组在所有设置范围内合并。还与 `Edit(...)` 允许权限规则中的路径合并。前缀：`/`（绝对路径）、`~/`（主目录）、`./` 或无（项目设置中为项目相对路径，用户设置中为 `~/.claude` 相对路径）。旧版 `//` 前缀表示绝对路径仍然有效。**注意：** 这与 [Read/Edit 权限规则](#工具权限语法)不同，后者使用 `//` 表示绝对路径，`/` 表示项目相对路径 |
| `sandbox.filesystem.denyWrite` | array | `[]` | 沙箱命令无法写入的路径。数组在所有设置范围内合并。还与 `Edit(...)` 拒绝权限规则中的路径合并。与 `allowWrite` 相同的路径前缀约定 |
| `sandbox.filesystem.denyRead` | array | `[]` | 沙箱命令无法读取的路径。数组在所有设置范围内合并。还与 `Read(...)` 拒绝权限规则中的路径合并。与 `allowWrite` 相同的路径前缀约定 |
| `sandbox.filesystem.allowRead` | array | `[]` | 在 `denyRead` 区域内重新允许读取访问的路径。优先级高于 `denyRead`。数组在所有设置范围内合并。与 `allowWrite` 相同的路径前缀约定 |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **（仅托管）** 仅尊重托管设置中的 `allowRead` 路径。来自用户、项目和本地设置的 `allowRead` 条目将被忽略 |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | （仅 macOS）允许访问系统 TLS 信任（`com.apple.trustd.agent`）；降低安全性 |

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

## 插件

配置 Claude Code 插件和市场。

### 插件设置

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enabledPlugins` | object | 任意 | 启用/禁用特定插件 |
| `extraKnownMarketplaces` | object | 项目 | 添加自定义插件市场（通过 `.claude/settings.json` 团队共享） |
| `strictKnownMarketplaces` | array | 仅托管 | 允许的市场列表 |
| `skippedMarketplaces` | array | 任意 | 用户拒绝安装的市场（在 JSON schema 中，不在官方设置页面） |
| `skippedPlugins` | array | 任意 | 用户拒绝安装的插件（在 JSON schema 中，不在官方设置页面） |
| `pluginConfigs` | object | 任意 | 每个插件的 MCP 服务器配置（以 `plugin@marketplace` 为键）（在 JSON schema 中，不在官方设置页面） |
| `blockedMarketplaces` | array | 仅托管 | 阻止特定插件市场。每个条目可以匹配 source 字符串、`hostPattern` 或 `pathPattern` —— 截至 v2.1.119，`hostPattern` 和 `pathPattern` 匹配器在任何内容接触文件系统之前被正确执行，因此被阻止的市场永远不会到达磁盘 |
| `pluginTrustMessage` | string | 仅托管 | 提示用户信任插件时显示的自定义消息 |

**市场来源类型：** `github`、`git`、`directory`、`hostPattern`、`settings`、`url`、`npm`、`file`。使用 `source: 'settings'` 可以在不设置托管市场仓库的情况下内联声明少量插件。

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

## 模型配置

### 模型别名

| 别名 | 描述 |
|-------|-------------|
| `"default"` | 推荐用于你的账户类型 |
| `"sonnet"` | 最新 Sonnet 模型（Claude Sonnet 4.6） |
| `"opus"` | 最新 Opus 模型（Claude Opus 4.6） |
| `"haiku"` | 快速 Haiku 模型 |
| `"sonnet[1m]"` | 带 1M token Context 的 Sonnet |
| `"opus[1m]"` | 带 1M token Context 的 Opus（自 v2.1.75 起 Max、Team 和 Enterprise 的默认值） |
| `"opusplan"` | 规划时使用 Opus，执行时使用 Sonnet |

### 模型覆盖

将 Anthropic 模型 ID 映射到 Bedrock、Vertex 或 Foundry 部署的特定于提供商的模型 ID。

### 努力级别

`/model` 命令提供**努力级别**控制，调整模型在每个回复中应用的推理量。在 `/model` UI 中使用 ← → 箭头键循环遍历努力级别。

| 努力级别 | 描述 |
|-------------|-------------|
| Max | 最大推理深度，仅 Opus 4.6 |
| XHigh | 扩展高推理深度，仅 Opus 4.7（v2.1.111 起 Opus 4.7 跨所有计划的默认值） |
| High（Opus 4.6/Sonnet 4.6 的默认值） | 完整推理深度，最适合复杂任务 |
| Medium | 平衡推理，适合日常任务 |
| Low | 最小推理，最快响应 |

### 模型环境变量

通过 `env` 键配置：

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

## 显示与 UX

### 显示设置

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `statusLine` | object | - | Custom status line configuration |
| `outputStyle` | string | `"default"` | Output style (e.g., `"Explanatory"`) |
| `spinnerTipsEnabled` | boolean | `true` | Show tips while waiting |
| `spinnerVerbs` | object | - | Custom spinner verbs with `mode` ("append" or "replace") and `verbs` array |
| `spinnerTipsOverride` | object | - | Custom spinner tips with `tips` (string array) and optional `excludeDefault` (boolean) |
| `respectGitignore` | boolean | `true` | Respect .gitignore in file picker |
| `prefersReducedMotion` | boolean | `false` | Reduce animations and motion effects in the UI |
| `fileSuggestion` | object | - | Custom file suggestion command (see File Suggestion Configuration below) |

### 全局配置设置（`~/.claude.json`）

These display preferences are stored in `~/.claude.json`, **not** `settings.json`. Adding them to `settings.json` will trigger a schema validation error.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `autoConnectIde` | boolean | `false` | Automatically connect to a running IDE when Claude Code starts from an external terminal. Appears in `/config` as **Auto-connect to IDE (external terminal)** when running outside a VS Code or JetBrains terminal |
| `autoInstallIdeExtension` | boolean | `true` | Automatically install the Claude Code IDE extension when running from a VS Code terminal. Appears in `/config` as **Auto-install IDE extension**. Can also be disabled via `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` env var |
| `autoScrollEnabled` | boolean | `true` | Auto-scroll the conversation in fullscreen mode. Set to `false` to disable automatic scrolling (v2.1.110) |
| `editorMode` | string | `"normal"` | Key binding mode for the input prompt: `"normal"` or `"vim"`. Appears in `/config` as **Editor mode** |
| `externalEditorContext` | boolean | `true` | Include additional context about the external editor when available. Set to `false` to disable |
| `showTurnDuration` | boolean | `true` | Show turn duration messages after responses (e.g., "Cooked for 1m 6s"). Edit `~/.claude.json` directly to change |
| `terminalProgressBarEnabled` | boolean | `true` | Show the terminal progress bar in supported terminals (ConEmu, Ghostty 1.2.0+, and iTerm2 3.6.6+). Appears in `/config` as **Terminal progress bar** |
| `teammateMode` | string | `"auto"` | How [agent team](https://code.claude.com/docs/en/agent-teams) teammates display: `"auto"` (picks split panes in tmux or iTerm2, in-process otherwise), `"in-process"`, or `"tmux"`. See [choose a display mode](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) |

### 状态栏配置

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
| `worktree.name` | Name of the active worktree (present only during `--worktree` sessions) |
| `worktree.path` | Absolute path to the worktree directory |
| `worktree.branch` | Git branch name for the worktree. Absent for hook-based worktrees |
| `worktree.original_cwd` | Directory before entering the worktree |
| `worktree.original_branch` | Git branch checked out before entering the worktree. Absent for hook-based worktrees |

### 文件建议配置

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

## AWS 与云凭据

### AWS 设置

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

## 环境变量（通过 `env` 设置）

为所有 Claude Code 会话设置环境变量。

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### 常见环境变量

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key for authentication |
| `ANTHROPIC_AUTH_TOKEN` | OAuth token |
| `CLAUDE_CODE_OAUTH_TOKEN` | OAuth access token for Claude.ai authentication. Alternative to `/login` for SDK and automated environments. Takes precedence over keychain-stored credentials |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | OAuth refresh token for Claude.ai authentication. When set, `claude auth login` exchanges this token directly instead of opening a browser. Requires `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | Space-separated OAuth scopes the refresh token was issued with (e.g., `"user:profile user:inference user:sessions:claude_code"`). Required when `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` is set |
| `ANTHROPIC_BASE_URL` | Custom API endpoint |
| `ANTHROPIC_BEDROCK_BASE_URL` | Override Bedrock endpoint URL |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | Override the Bedrock Mantle endpoint URL. See [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
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
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | Set to `1` to allow fast mode when the organization status check fails due to a network error. Useful when a corporate proxy blocks the status endpoint |
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock (`1` to enable) |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex AI (`1` to enable) |
| `CLAUDE_CODE_USE_FOUNDRY` | Use Microsoft Foundry (`1` to enable) |
| `CLAUDE_CODE_USE_MANTLE` | Use the Bedrock [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) (`1` to enable) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Set to `1` to enable the PowerShell tool on Windows (opt-in preview). When enabled, Claude can run PowerShell commands natively instead of routing through Git Bash. Only supported on native Windows, not WSL (v2.1.84) |
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
| `CLAUDE_CODE_MAX_TURNS` | Maximum agentic turns before stopping *(not in official docs — unverified)* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Equivalent of setting `DISABLE_AUTOUPDATER`, `DISABLE_FEEDBACK_COMMAND`, `DISABLE_ERROR_REPORTING`, and `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | Skip first-run settings setup flow *(not in official docs — unverified)* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | Override prompt caching behavior *(not in official docs — unverified)* |
| `CLAUDE_CODE_DISABLE_TOOLS` | Comma-separated list of tools to disable *(not in official docs — unverified)* |
| `CLAUDE_CODE_DISABLE_MCP` | Disable all MCP servers (`1` to disable) *(not in official docs — unverified)* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens per response. Default: 32,000 (64,000 for Opus 4.6 as of v2.1.77). Upper bound: 64,000 (128,000 for Opus 4.6 and Sonnet 4.6 as of v2.1.77) |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | Disable fast mode entirely (`1` to disable) |
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
| `DISABLE_EXTRA_USAGE_COMMAND` | Hide the `/extra-usage` command (`1` to disable) |
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
| `CLAUDE_CODE_DISABLE_MOUSE` | Set to `1` to disable mouse tracking in fullscreen rendering. Useful when mouse events interfere with terminal multiplexers or accessibility tools |
| `CLAUDE_CODE_HIDE_CWD` | Set to `1` to hide the current working directory in the Claude Code startup logo banner. Useful in screen recordings, demos, or shared sessions where the CWD path leaks information about the host or project layout (v2.1.119) |
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

## 常用命令

| 命令 | 描述 |
|---------|-------------|
| `/model` | 切换模型并调整 Opus 4.6 努力级别 |
| `/effort` | 直接设置努力级别：`low`、`medium`、`high`（v2.1.76+） |
| `/config` | 交互式配置 UI |
| `/memory` | 查看/编辑所有记忆文件 |
| `/agents` | 管理 subagents |
| `/mcp` | 管理 MCP 服务器 |
| `/hooks` | 查看已配置的 hooks |
| `/plugin` | 管理插件 |
| `claude plugin tag` | 在市场中标记插件版本以便分发。从市场仓库中运行，带上插件名称和版本（v2.1.118） |
| `/keybindings` | 配置自定义键盘快捷键 |
| `/skills` | 查看和管理 skills |
| `/permissions` | 查看和管理权限规则 |
| `--doctor` | 诊断配置问题 |
| `--debug` | 调试模式，包含 hook 执行详情 |

---

## 快速参考：完整示例

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
  "effortLevel": "xhigh",

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

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "xhigh"
  }
}
```

---

## 来源

- [Claude Code Settings 文档](https://code.claude.com/docs/en/settings)
- [Claude Code Settings JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code 变更日志](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub Settings 示例](https://github.com/feiskyer/claude-code-settings)
- [Shipyard - Claude Code CLI 速查表](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Claude Code 环境变量参考](https://code.claude.com/docs/en/env-vars)
- [Claude Code 权限参考](https://code.claude.com/docs/en/permissions)

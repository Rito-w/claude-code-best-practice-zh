# Settings 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Jun%2011%2C%202026%2010%3A43%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.172-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../.claude/settings.json)

本指南全面介绍 Claude Code `settings.json` 文件中所有可用的配置选项。截至 v2.1.172，Claude Code 开放了 **80+ 设置项** 和 **200+ 环境变量**（使用 `settings.json` 中的 `"env"` 字段可避免编写包装脚本）。

<table width="100%">
<tr>
<td><a href="..//">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 目录

1. [设置层级](#settings-hierarchy)
2. [核心配置](#core-configuration)
3. [权限](#permissions)
4. [钩子](#hooks)
5. [MCP 服务器](#mcp-servers)
6. [沙盒](#sandbox)
7. [插件](#plugins)
8. [模型配置](#model-configuration)
9. [显示与用户体验](#display--ux)
10. [AWS 与云凭据](#aws--cloud-credentials)
11. [环境变量（通过 env）](#environment-variables-via-env)
12. [实用命令](#useful-commands)

---

## 设置层级

设置按优先级顺序生效（从高到低）：

| 优先级 | 位置 | 范围 | 共享？ | 用途 |
|----------|----------|-------|---------|---------|
| 1 | 托管设置 | 组织 | 是（由 IT 部署） | 不可被覆盖的安全策略 |
| 2 | 命令行参数 | 会话 | N/A | 临时单次会话覆盖 |
| 3 | `.claude/settings.local.json` | 项目 | 否（被 git 忽略） | 个人项目专用 |
| 4 | `.claude/settings.json` | 项目 | 是（已提交） | 团队共享设置 |
| 5 | `~/.claude/settings.json` | 用户 | N/A | 全局个人默认设置 |

**托管设置** 由组织强制执行，无法被任何其他层级覆盖，包括命令行参数。交付方式：
- **服务器托管** 设置（远程交付）
- **MDM 配置文件** — macOS plist 位于 `com.anthropic.claudecode`
- **注册表策略** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode`（管理员）和 `HKCU\SOFTWARE\Policies\ClaudeCode`（用户级，最低策略优先级）
- **文件** — `managed-settings.json` 和 `managed-mcp.json`（macOS: `/Library/Application Support/ClaudeCode/`，Linux/WSL: `/etc/claude-code/`，Windows: `C:\Program Files\ClaudeCode\`）
- **Drop-in 目录** — `managed-settings.d/` 与 `managed-settings.json` 并列，用于独立的策略片段（v2.1.83）。遵循 systemd 约定，`managed-settings.json` 首先作为基础合并，然后 drop-in 目录中所有 `*.json` 文件按字母顺序排序并合并。后面的文件覆盖前面的文件：标量值被覆盖，数组被连接并去重，对象被深度合并。以 `.` 开头的隐藏文件被忽略。使用数字前缀控制合并顺序（例如 `10-telemetry.json`、`20-security.json`）

在托管层级内，优先级为：服务器托管 > MDM/操作系统级策略 > 基于文件的（`managed-settings.d/*.json` + `managed-settings.json`）> HKCU 注册表（仅 Windows）。仅使用一个托管来源；各层级之间不合并。在基于文件的层级内，drop-in 文件和基础文件会被合并。

> **注意：** 截至 v2.1.75，已弃用的 Windows 回退路径 `C:\ProgramData\ClaudeCode\managed-settings.json` 已被移除。请改用 `C:\Program Files\ClaudeCode\managed-settings.json`。

> **注意（v2.1.126）：** `/config` 现在将更改持久化到 `~/.claude/settings.json`，而不再仅保存在内存中。通过交互式配置 UI 进行的编辑在重启后依然保留。

**仅限托管的策略键：**

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `parentSettingsBehavior` | string | `"first-wins"` | 控制当存在管理员部署的托管层级时，由嵌入宿主进程（SDK 父进程）以编程方式提供的托管设置是否生效。`"first-wins"`：丢弃父进程提供的设置，仅应用管理员层级。`"merge"`：父进程提供的设置在管理员层级之下生效，并被过滤以确保只能**收紧**策略而不能放松。需要 v2.1.133+ |
| `policyHelper` | object | - | 管理员部署的可执行文件，在启动时动态计算托管设置。对象结构：`{path: string}` 指向辅助程序二进制文件。仅从 MDM 或系统级 `managed-settings.json` 文件中接受（绝不会从用户/项目设置中接受）。辅助程序输出在每次启动时合并到托管层级。需要 v2.1.136+ |
| `requiredMinimumVersion` | string | - | **（仅限托管）** 如果已安装版本低于此下限，则阻止 Claude Code 启动。CLI 会退出并显示错误提示用户升级。补充 `minimumVersion`（控制自动更新下限）— 此项在启动时强制执行。示例：`"2.1.163"` |
| `requiredMaximumVersion` | string | - | **（仅限托管）** 如果已安装版本超过此上限，则阻止 Claude Code 启动。如果版本太新，CLI 会退出并显示错误。与 `requiredMinimumVersion` 配合使用可在托管环境中固定特定版本范围。示例：`"2.1.165"` |

**重要说明**：
- `deny` 规则具有最高安全优先级，不能被低优先级的 allow/ask 规则覆盖。
- 托管设置可能锁定或覆盖本地行为，即使本地文件指定了不同的值。
- 数组设置（如 `permissions.allow`）在各范围之间**合并并去重** — 所有层级的条目被组合在一起，而非替换。

---

## 核心配置

### 通用设置

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `$schema` | string | - | JSON Schema URL，用于 IDE 验证和自动补全（如 `"https://json.schemastore.org/claude-code-settings.json"`） |
| `model` | string | `"default"` | 覆盖默认模型。接受别名（`sonnet`、`opus`、`haiku`）或完整模型 ID |
| `agent` | string | - | 设置主对话的默认 agent。值为 `.claude/agents/` 中的 agent 名称。也可通过 `--agent` CLI 标志使用 |
| `language` | string | `"english"` | Claude 的首选响应语言。同时设置语音听写语言和终端标签页标题（v2.1.121） |
| `claudeMdExcludes` | array | - | 加载 [memory](https://code.claude.com/docs/en/memory) 时要跳过的 `CLAUDE.md` 文件的 glob 模式或绝对路径。模式匹配绝对文件路径。仅适用于用户、项目和本地 memory；无法排除托管策略文件。示例：`["**/vendor/**/CLAUDE.md"]` |
| `claudeMd` | string | - | **（仅限托管）** CLAUDE.md 风格的指令，作为组织托管的 [memory](https://code.claude.com/docs/en/memory) 注入。仅在托管或策略设置中生效；在用户、项目和本地设置中被忽略。示例：`"Always run make lint before committing."` |
| `cleanupPeriodDays` | number | `30` | 启动清理扫掠的截止天数（最小值 1）。删除不活跃的会话记录和孤立的子 agent 工作树；截至 v2.1.117，扫掠还覆盖 `~/.claude/tasks/`、`~/.claude/shell-snapshots/` 和 `~/.claude/backups/`。设置为 `0` 会被验证错误拒绝。要在非交互模式（`-p`）下禁止记录写入，请使用 `--no-session-persistence` 或 `persistSession: false` SDK 选项 |
| `autoUpdatesChannel` | string | `"latest"` | 发布通道：`"stable"` 或 `"latest"` |
| `minimumVersion` | string | - | 防止自动更新程序降级到指定版本以下。切换到稳定通道并选择停留在当前版本直到稳定版本追平时自动设置。与 `autoUpdatesChannel` 配合使用 |
| `alwaysThinkingEnabled` | boolean | `false` | 为所有会话默认启用扩展思考 |
| `skipWebFetchPreflight` | boolean | `false` | 跳过 WebFetch 域名安全检查（该检查会在获取前将每个请求的主机名发送到 `api.anthropic.com`）。在阻止出站流量到 Anthropic 的环境中设为 `true`，例如 Bedrock、Vertex AI 或具有严格出口限制的 Foundry 部署 |
| `availableModels` | array | - | 限制用户可通过 `/model`、`--model`、配置工具或 `ANTHROPIC_MODEL` 选择的模型。不影响默认选项。示例：`["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | 要求用户每个会话手动选择加入快速模式 |
| `defaultShell` | string | `"bash"` | 输入框 `!` 命令的默认 shell。接受 `"bash"`（默认）或 `"powershell"`。设置 `"powershell"` 会在 Windows 上将交互式 `!` 命令路由到 PowerShell。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`（v2.1.84）。**v2.1.120：** 当 PowerShell 可用时，即使未安装 Git for Windows，它也用作 Windows 上的回退 shell。**v2.1.126：** 启用 PowerShell 后，它被视为*主要* shell，而非默认使用 Bash。PowerShell 7 检测现在还覆盖 Microsoft Store 安装、不在 PATH 中的 MSI 安装和 `.NET` 全局工具安装 |
| `includeGitInstructions` | boolean | `true` | 在 Claude 的系统提示中包含内置的提交和 PR 工作流指令以及 git 状态快照。当设置 `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` 环境变量时，该变量优先于此设置 |
| `voice` | object | - | 语音听写配置。包含三个字段的对象：`enabled`（布尔值 — 按键通话开/关）、`mode`（字符串 — `"hold"` 为按住说话或 `"tap"` 为点击切换）和 `autoSubmit`（布尔值 — 听写结束时立即发送文本）。运行 `/voice` 时自动写入。需要 Claude.ai 账户（v2.1.118 扩展结构） |
| `voiceEnabled` | boolean | - | **已弃用** — `voice.enabled` 的旧别名。请改用 `voice` 对象以获取 `mode` 和 `autoSubmit` 控制 |
| `showClearContextOnPlanAccept` | boolean | `false` | 在计划接受屏幕上显示"清除上下文"选项。设为 `true` 以恢复该选项（自 v2.1.81 起默认隐藏） |
| `viewMode` | string | - | 启动时的默认记录视图模式：`"default"`、`"verbose"` 或 `"focus"`。设置后会覆盖粘性 Ctrl+O 选择 |
| `disableDeepLinkRegistration` | string | - | 设置为 `"disable"` 以防止 Claude Code 在启动时将 `claude-cli://` 协议处理程序注册到操作系统。深层链接允许外部工具通过 `claude-cli://open?q=...` 打开预填提示的 Claude Code 会话。`q` 参数支持使用 URL 编码的换行符（`%0A`）的多行提示。在协议处理程序注册受限或单独管理的环境中有用 |
| `showThinkingSummaries` | boolean | `false` | 在交互会话中显示扩展思考摘要。未设置或为 `false` 时（交互模式下的默认值），思考块会被 API 编辑并以折叠的摘要形式显示。编辑仅改变你看到的内容，不改变模型生成的内容 — 要减少思考开销，请降低预算或禁用思考。非交互模式（`-p`）和 SDK 调用者始终会收到摘要，无论此设置如何 |
| `disableSkillShellExecution` | boolean | `false` | 禁止来自用户、项目、插件或额外目录来源的技能和自定义命令中的内联 shell 执行（`` !`...` `` 和 `` ```! `` 块）。命令会被替换为 `[shell command execution disabled by policy]` 而非执行。内置和托管技能不受影响（v2.1.91） |
| `maxSkillDescriptionChars` | number | `1536` | 每个技能的 `description` 和 `when_to_use` 文本合并后的字符上限，用于 Claude 每轮看到的 [技能列表](https://code.claude.com/docs/en/skills)。超出此长度的文本会被截断（v2.1.105） |
| `skillListingBudgetFraction` | number | `0.01` | 为 Claude 每轮看到的 [技能列表](https://code.claude.com/docs/en/skills) 预留的模型上下文窗口比例（`0.01` = 1%）。当列表超出预算时，使用频率最低的技能描述会被折叠为仅显示名称，Claude 仍能调用它们但看不到说明（v2.1.105） |
| `forceRemoteSettingsRefresh` | boolean | `false` | **（仅限托管）** 阻止 CLI 启动，直到远程托管设置被重新获取。如果获取失败，CLI 会退出（故障关闭）。用于必须在会话开始前确保策略最新的企业环境（v2.1.92） |
| `wslInheritsWindowsSettings` | boolean | `false` | **（仅限 Windows 托管设置）** 当为 `true` 时，WSL 上的 Claude Code 会从 Windows 策略链（HKLM 注册表 + `C:\Program Files\ClaudeCode\managed-settings.json`）以及 `/etc/claude-code` 读取托管设置，Windows 来源优先。仅在 HKLM 注册表键或 `C:\Program Files\ClaudeCode\managed-settings.json` 中设置时才生效，两者都需要 Windows 管理员权限写入。要让 HKCU 策略也在 WSL 上生效，还必须在 HKCU 中设置此标志。对原生 Windows 无效（v2.1.118） |
| `tui` | string | `"default"` | 渲染模式：`"fullscreen"` 或 `"default"`。通过 `/tui fullscreen` 设置以进行无闪烁的备用屏幕渲染（v2.1.110） |
| `awaySummaryEnabled` | boolean | `true` | 用户离开后返回时生成"离开摘要"（空闲会话回顾）。设为 `false` 以退出。与 `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` 环境变量配合使用（v2.1.110） |
| `skillOverrides` | object | - | 按技能名称键入的每个技能可见性覆盖。值为 `"on"`（完整）、`"name-only"`（可见但不自动描述）、`"user-invocable-only"`（对模型发现隐藏但仍可通过斜杠调用）或 `"off"`（完全隐藏）。示例：`{"legacy-context": "name-only", "deploy": "off"}`（v2.1.129） |
| `disableRemoteControl` | boolean | `false` | 禁用 [远程控制](https://code.claude.com/docs/en/remote-control)：阻止 `claude remote-control`、`--remote-control` 标志、自动启动和会话内切换。通常放置在托管设置中以进行每设备 MDM 强制执行，但也适用于任何范围（v2.1.128） |
| `disableAgentView` | boolean | `false` | 设为 `true` 以关闭 [后台 agent 和 agent 视图](https://code.claude.com/docs/en/agent-view)：`claude agents`、`--bg`、`/background` 和按需监督器。可在任何范围设置，但通常放置在托管设置中。等效于将 `CLAUDE_CODE_DISABLE_AGENT_VIEW` 环境变量设置为 `1` |
| `disableWorkflows` | boolean | `false` | 设为 `true` 以禁用 [动态工作流](https://code.claude.com/docs/en/workflows)（`/workflows`）和内置的工作流斜杠命令。可在任何范围设置。等效于 `CLAUDE_CODE_DISABLE_WORKFLOWS` 环境变量。工作流于 v2.1.154 引入 |
| `workflowKeywordTriggerEnabled` | boolean | `true` | 在提示中输入单词 "ultracode" 是否触发 [动态工作流](https://code.claude.com/docs/en/workflows)。设为 `false` 以要求显式 `/workflows` 调用。Ultracode、`/workflows` 和已保存的工作流命令不受此设置影响。在 `/config` 中显示为 **Workflow keyword trigger**（v2.1.157；触发词在 v2.1.160 中从 workflow 改名为 ultracode） |
| `ultracode` | boolean | - | **（仅会话 — 不持久化）** 当为 `true` 时，Harness 默认为每个实质性任务编写并运行工作流，以最大化彻底性而不考虑 token 成本。出现在官方"可用设置"列表中，但仅作用域于会话：通过 `/effort ultracode`、`--settings` 或 SDK 设置，而非写入 `settings.json`（v2.1.154） |
| `disableBundledSkills` | boolean | `false` | 隐藏 Claude Code 的内置能力（bundled skills）。设为 `true` 时，模型无法调用内置技能。与 `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` 环境变量配对使用。适用于需要严格插件定制的场景 *(v2.1.170 changelog 中，尚未出现在官方设置页面)* |
| `feedbackSurveyRate` | number | - | 当符合条件时出现会话质量调查的概率（0–1）。企业管理员可控制调查展示频率。示例：`0.05` = 5% 的合格会话 |
| `advisorModel` | string | - | 服务器端 advisor 工具使用的模型。接受模型别名（`opus`、`sonnet`、`fable`）或完整模型 ID。未设置时，advisor 使用会话模型。需要 v2.1.98+ |

**示例：**
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

### 计划与 Memory 目录

将计划和自动 memory 文件存储在自定义位置。

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | `/plan` 输出的存储目录 |
| `autoMemoryDirectory` | string | - | 自动 memory 存储的自定义目录。接受 `~/` 展开的路径。在项目设置（`.claude/settings.json`）中不接受，以防止将 memory 写入重定向到敏感位置；在策略、本地和用户设置中接受 |
| `autoMemoryEnabled` | boolean | `true` | 启用 [自动 memory](https://code.claude.com/docs/en/memory)。当为 `false` 时，Claude 不会读取或写入自动 memory 目录。也可在会话期间通过 `/memory` 切换，或通过 `CLAUDE_CODE_DISABLE_AUTO_MEMORY` 环境变量禁用 |

**示例：**
```json
{
  "plansDirectory": "./my-plans"
}
```

**用途：** 可用于将规划产物与 Claude 的内部文件分开管理，或将计划保存在团队共享位置。

### Worktree 设置

配置 `--worktree` 如何创建和管理 git worktree。在大型 monorepo 中减少磁盘使用和启动时间。

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | 要从主仓库符号链接到每个 worktree 的目录，以避免在磁盘上复制大型目录 |
| `worktree.sparsePaths` | array | `[]` | 通过 git sparse-checkout（cone 模式）在每个 worktree 中检出的目录。只有列出的路径会被写入磁盘 |
| `worktree.baseRef` | string | `"fresh"` | 新 worktree 分支自哪个 ref。`"fresh"` 从 `origin/<default-branch>` 分支，创建与远程匹配的干净树。`"head"` 从当前本地 `HEAD` 分支，包含未提交但已跟踪的更改（v2.1.133） |
| `worktree.bgIsolation` | string | `"worktree"` | [后台会话](https://code.claude.com/docs/en/agent-view) 的隔离模式。`"worktree"`（默认）在主检出中阻止 `Edit`/`Write`，直到调用 `EnterWorktree`；`"none"` 允许后台作业直接编辑工作副本（v2.1.143） |

**示例：**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### 归属设置

自定义 git 提交和拉取请求的归属消息。

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `attribution.commit` | string | Co-authored-by | Git 提交归属（支持 trailer） |
| `attribution.pr` | string | 生成的消息 | 拉取请求描述归属 |
| `prUrlTemplate` | string | - | 控制提交归属中 "PR" 徽章如何链接到拉取请求 UI 的 URL 模板。支持仓库主机、所有者、仓库和 PR 编号的占位符。适用于自托管的 GitLab/Bitbucket/GitHub Enterprise 实例，其中默认的 `https://github.com/...` URL 不适用（v2.1.119） |
| `includeCoAuthoredBy` | boolean | `true` | **已弃用** - 请改用 `attribution` |

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

### 认证助手

用于动态认证 token 生成的脚本。

| 键 | 类型 | 描述 |
|-----|------|-------------|
| `apiKeyHelper` | string | 输出认证 token 的 shell 脚本路径（作为 `X-Api-Key` 请求头发送） |
| `forceLoginMethod` | string | 限制登录为 `"claudeai"` 或 `"console"` 账户 |
| `forceLoginOrgUUID` | string \| array | 要求登录属于特定组织。接受单个 UUID 字符串（同时会在登录时预选该组织）或 UUID 数组，其中列出的任何组织都被接受而不预选。在托管设置中设置时，如果认证账户不属于列出的组织则登录失败；空数组会以配置错误消息关闭登录 |
| `gcpAuthRefresh` | string | 当 GCP Application Default Credentials 过期或无法加载时刷新它的自定义脚本。由 Claude Code 在重试认证前运行。适用于 ADC 是短期凭据且需要组织特定助手来续期的场景。示例：`"gcloud auth application-default login"` |

**示例：**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### 公司公告

在启动时向用户显示自定义公告（随机循环播放）。

| 键 | 类型 | 描述 |
|-----|------|-------------|
| `companyAnnouncements` | array | 启动时显示的字符串数组 |

**示例：**
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

| 键 | 类型 | 描述 |
|-----|------|-------------|
| `permissions.allow` | array | 允许使用工具而不提示的规则 |
| `permissions.ask` | array | 需要用户确认的规则 |
| `permissions.deny` | array | 阻止使用工具的规则（最高优先级） |
| `permissions.additionalDirectories` | array | Claude 可访问的额外目录 |
| `permissions.defaultMode` | string | 默认权限模式。在远程环境中，仅 `acceptEdits` 和 `plan` 被认可（v2.1.70+） |
| `permissions.disableBypassPermissionsMode` | string | 阻止绕过权限模式激活 |
| `permissions.skipDangerousModePermissionPrompt` | boolean | 跳过通过 `--dangerously-skip-permissions` 或 `defaultMode: "bypassPermissions"` 进入绕过权限模式前显示的确认提示。在项目设置（`.claude/settings.json`）中设置时被忽略，以防止不受信任的仓库自动绕过提示 |
| `allowManagedPermissionRulesOnly` | boolean | **（仅限托管）** 仅应用托管权限规则；用户/项目的 `allow`、`ask`、`deny` 规则被忽略 |
| `autoMode` | object | 自定义 [自动模式](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 分类器阻止和允许的内容。包含 `environment`（受信任的基础设施描述）、`allow`（阻止规则的例外）、`soft_deny`（阻止规则）和 `hard_deny`（无条件阻止规则 — 不能被 `allow` 例外或 `$defaults` 哨兵覆盖，v2.1.136）— 均为散文字符串数组。**不从共享项目设置**（`.claude/settings.json`）中读取，以防止仓库注入。可在用户、本地和托管设置中使用。设置 `allow` 或 `soft_deny` 会**替换**该部分的整个默认列表，除非你在数组中包含字面字符串 `"$defaults"` — 该哨兵在该位置继承内置规则，以便自定义条目与之并列添加（v2.1.118）。运行 `claude auto-mode_defaults` 可在自定义前查看内置规则 |
| `disableAutoMode` | string | 设置为 `"disable"` 以防止 [自动模式](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 被激活。从 `Shift+Tab` 循环中移除 `auto` 并在启动时拒绝 `--permission-mode auto`。可在任何设置层级设置；在托管设置中最有用，因为用户无法覆盖 |
| `useAutoModeDuringPlan` | boolean | 当自动模式可用时，计划模式是否使用自动模式语义。默认：`true`。不从共享项目设置（`.claude/settings.json`）中读取。在 `/config` 中显示为 "Use auto mode during plan" |

### 权限模式

| 模式 | 行为 |
|------|----------|
| `"default"` | 标准权限检查，带提示 |
| `"acceptEdits"` | 自动接受文件编辑**和常见文件系统命令**（`mkdir`、`touch`、`mv`、`cp` 等），适用于工作目录或 `additionalDirectories` 中的路径。**v2.1.160：** 在写入授予代码执行权限的构建工具配置文件（`.npmrc`、`.yarnrc*`、`bunfig.toml`、`.bazelrc`、`.pre-commit-config.yaml`、`.devcontainer/` 等）以及写入 shell 启动文件（`.zshenv`、`.zlogin`、`.bash_login`）和 `~/.config/git/` 之前始终会提示 |
| `"dontAsk"` | 自动拒绝工具，除非已通过 `/permissions` 或 `permissions.allow` 规则预先批准 |
| `"bypassPermissions"` | 跳过所有权限检查（危险）。跳过所有基于路径的提示 — 写入 `.git`、`.config/git`、`.claude`、`.vscode`、`.idea`、`.husky`、`.cargo`、`.devcontainer`、`.yarn` 和 `.mvn` 不再提示（**v2.1.121** 豁免了 `.claude/commands/`、`.claude/agents/`、`.claude/skills/` 和 `.claude/worktrees/`；**v2.1.126** 移除了所有剩余的基于路径的提示）。仅针对文件系统根目录或主目录的删除操作（`rm -rf /`、`rm -rf ~`）仍会提示，作为防止模型错误的保险丝 |
| `"auto"` | 自动批准工具调用，附带后台安全检查以验证操作是否符合你的请求。研究预览。分类器自动批准只读和文件编辑；将其他所有内容发送安全检查。连续 3 次或总计 20 次阻止后回退到提示。自 v2.1.111 起在默认 `Shift+Tab` 权限模式循环中（`--enable-auto-mode` 标志在 v2.1.111 中已移除 — 使用 `--permission-mode auto` 启动）。使用 `autoMode` 设置配置 |
| `"plan"` | 只读探索模式。截至 v2.1.136，即使存在匹配的 `Edit(...)` 允许规则，文件写入也会被阻止 — 计划模式现在覆盖显式允许规则以维持其只读保证 |

### 工具权限语法

| 工具 | 语法 | 示例 |
|------|--------|----------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`、`Bash(* install)`、`Bash(git * main)` |
| `PowerShell` | `PowerShell(cmd *)` | `PowerShell(Get-ChildItem *)`、`PowerShell(git commit *)` — 与 Bash 形状相同；常用别名会被规范化（`gci`/`ls`/`dir` → `Get-ChildItem`），并解析 PowerShell AST，因此 `|`/`;`/`&&`/`||` 链中的每个子命令都必须匹配 |
| `Read` | `Read(path pattern)` | `Read(.env)`、`Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`、`Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`、`Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | 全局网络搜索 |
| `Task` | `Task(agent-name)` | `Task(Explore)`、`Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`、`Agent(*)` — 作用域限于子 agent 生成 |
| `Skill` | `Skill(skill-name)` 或 `Skill(prefix *)` | `Skill(weather-fetcher)`、`Skill(weather *)` 匹配 `weather-fetcher`/`weather-svg-creator`（v2.1.139） |
| `MCP` | `mcp__server__tool` 或 `MCP(server:tool)` | `mcp__memory__*`、`MCP(github:*)` |
| `Cd` | `Cd(path pattern)` | `Cd(/home/*)`、`Cd(~/projects/*)` — 控制 `/cd` 命令可以导航到哪些目录 |

**评估顺序：** 规则按以下顺序评估：首先是 deny 规则，然后是 ask，最后是 allow。第一个匹配的规则生效。

**Deny 规则 glob 模式（v2.1.166）：** 在 `deny` 规则中，在工具名称位置使用 `"*"` 匹配所有工具 — 等效于全局 deny。例如，deny 数组中的 `"*"` 会阻止每个工具调用。这使得可以完全锁定访问权限并开出具体的 allow/ask 例外。

**读/编辑路径模式：** `Read`、`Edit` 和 `Write` 的权限规则支持 gitignore 风格的模式，带有四种前缀类型：

| 前缀 | 含义 | 示例 |
|--------|---------|---------|
| `//` | 从文件系统根目录开始的绝对路径 | `Read(//Users/alice/file)` |
| `~/` | 相对于主目录 | `Read(~/.zshrc)` |
| `/` | 相对于项目根目录 | `Edit(/src/**)` |
| `./` 或无前缀 | 相对路径（当前目录） | `Read(.env)`、`Read(*.ts)` |

**符号链接解析：** 权限规则会检查符号链接路径及其解析目标。**允许**规则仅在符号链接*和*其目标都匹配时才应用 — 允许目录内指向外部的符号链接仍会提示。**拒绝**规则在符号链接*或*其目标任一匹配时应用 — 指向被拒绝文件的符号链接本身也被拒绝。

**Bash 通配符说明：**
- `*` 可出现在**任何位置**：前缀（`Bash(* install)`）、后缀（`Bash(npm *)`）或中间（`Bash(git * main)`）
- **词边界：** `Bash(ls *)`（`*` 前有空格）匹配 `ls -la` 但不匹配 `lsof`；`Bash(ls*)`（无空格）两者都匹配
- `Bash(*)` 被视为等效于 `Bash`（匹配所有 bash 命令）
- 权限规则支持输出重定向：`Bash(python:*)` 匹配 `python script.py > output.txt`
- 旧的 `:*` 后缀语法（如 `Bash(npm:*)`）等效于 ` *`，但已弃用
- **复合命令：** shell 运算符（`&&`、`||`、`;`、`|`、`|&`、`&` 和换行符）会分割命令，每个子命令必须独立匹配 — `Bash(safe-cmd *)` **不**授权 `safe-cmd && other-cmd`
- **进程包装器：** `timeout`、`time`、`nice`、`nohup` 和 `stdbuf` 在匹配前被剥离（因此 `Bash(npm test *)` 也匹配 `timeout 30 npm test`）；裸 `xargs`（无标志）也被剥离。执行包装器 `watch`、`setsid`、`ionice`、`flock` 和带 `-exec`/`-delete` 的 `find` 始终会提示，无法通过前缀规则批准

**示例：**
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

## 钩子

钩子配置（事件、属性、匹配器、退出码、环境变量和 HTTP 钩子）维护在一个专用仓库中：

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — 完整的钩子参考，包含声音通知系统、全部 25 个钩子事件、HTTP 钩子、匹配器模式、退出码和环境变量。

与钩子相关的设置键（`hooks`、`disableAllHooks`（也会禁用任何自定义状态行）、`allowManagedHooksOnly`、`allowedHttpHookUrls`、`httpHookAllowedEnvVars`）在其中有文档说明。

有关官方钩子参考，请参阅 [Claude Code 钩子文档](https://code.claude.com/docs/en/hooks)。

---

## MCP 服务器

配置 Model Context Protocol 服务器以扩展功能。

> **OAuth（v2.1.111）：** 通过 OAuth 认证的 MCP 服务器遵循 [RFC 9728](https://datatracker.ietf.org/doc/rfc9728/) 进行受保护资源元数据发现。合规服务器在 `/.well-known/oauth-protected-resource` 下公开授权端点，Claude Code 会自动完成 OAuth 流程 — 无需手动 `apiKeyHelper` 或 `headersHelper` 脚本用于符合规范的服务器。

> **保留服务器名称（v2.1.128）：** `workspace` 是保留的 MCP 服务器名称。加载时会跳过具有此名称的用户定义服务器，并在会话日志中记录警告。重命名任何预先存在的 `workspace` 服务器以避免冲突。

> **`.mcp.json` 热重载（v2.1.139）：** `/mcp` 的 Reconnect 操作现在会在重新连接前从磁盘重新读取 `.mcp.json`，因此添加或编辑服务器不再需要重启会话。Claude Code 还会将 `CLAUDE_PROJECT_DIR` 注入到通过 stdio 启动的 MCP 服务器环境中（v2.1.139），使服务器可以解析相对于项目根目录的路径。

> **每服务器超时下限（v2.1.162）：** 小于 1000ms 的每服务器 `timeout` 值会被忽略，并改用全局 `MCP_TOOL_TIMEOUT` 默认值。≥ 1000ms 的值照常生效。

### MCP 设置

| 键 | 类型 | 范围 | 描述 |
|-----|------|-------|-------------|
| `enableAllProjectMcpServers` | boolean | 任意 | 自动批准所有 `.mcp.json` 服务器 |
| `enabledMcpjsonServers` | array | 任意 | 允许列表，指定特定服务器名称 |
| `disabledMcpjsonServers` | array | 任意 | 阻止列表，指定特定服务器名称 |
| `allowedMcpServers` | array | 仅限托管 | 允许列表，支持名称/命令/URL 匹配 |
| `deniedMcpServers` | array | 仅限托管 | 阻止列表，支持匹配 |
| `allowManagedMcpServersOnly` | boolean | 仅限托管 | 仅允许托管允许列表中明确列出的 MCP 服务器 |
| `channelsEnabled` | boolean | 仅限托管 | 为 Team 和 Enterprise 用户允许 [channels](https://code.claude.com/docs/en/channels)。未设置或为 `false` 时，无论 `--channels` 标志如何，通道消息传递都会被阻止 |
| `allowedChannelPlugins` | array | 仅限托管 | 可推送消息的通道插件允许列表。设置时替换默认的 Anthropic 允许列表。未定义 = 回退到默认值，空数组 = 阻止所有通道插件。需要 `channelsEnabled: true`。每个条目是包含 `marketplace` 和 `plugin` 字段的对象（v2.1.84） |
| `allowAllClaudeAiMcps` | boolean | 仅限托管 | 加载 claude.ai 云 MCP 连接器以及 `managed-mcp.json`。启用时，claude.ai 托管的 MCP 连接器会与管理员部署的托管 MCP 服务器一起提供 |

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

### 每服务器工具加载（`alwaysLoad`，v2.1.121）

默认情况下，MCP 工具定义被延迟（通过工具搜索按需加载到上下文中）。在 `.mcp.json`（或内联 `mcpServers`）中的单个 MCP 服务器条目上设置 `alwaysLoad: true` 可使该服务器免于延迟 — 该服务器的每个工具都在会话启动时预先加载，无论 `ENABLE_TOOL_SEARCH` 如何。适用于所有服务器类型；需要 Claude Code v2.1.121+。仅对每轮都需要的一小部分工具使用此选项 — 每个预先加载的工具都会消耗原本可用于对话的上下文。

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

MCP 服务器也可以通过在工具的 `_meta` 对象中包含 `"anthropic/alwaysLoad": true` 来标记个别工具为始终加载 — 适用于只需要服务器部分工具绕过延迟的场景。

**示例：**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## 沙盒

配置 bash 命令沙盒以增强安全性。

### 沙盒设置

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `sandbox.enabled` | boolean | `false` | 启用 bash 沙盒 |
| `sandbox.failIfUnavailable` | boolean | `false` | 当沙盒已启用但无法启动时退出并报错，而非在不沙盒的情况下运行。适用于需要严格沙盒的企业策略（v2.1.83） |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | 沙盒化时自动批准 bash。截至 v2.1.139，shell 展开形式（`$VAR`、`$(cmd)`）被正确识别，因此包含变量替换的命令在沙盒自动批准时不再回退到提示 |
| `sandbox.excludedCommands` | array | `[]` | 在沙盒外运行的命令 |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | 允许 `dangerouslyDisableSandbox`。设置为 `false` 时，逃生出口被完全禁用，所有命令必须在沙盒中运行（或在 `excludedCommands` 中）。适用于需要严格沙盒的企业策略 |
| `sandbox.ignoreViolations` | object | `{}` | 命令模式到路径数组的映射 — 抑制违规警告 *（在 JSON schema 中，不在官方设置页面上）* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | **（仅限 Linux 和 WSL2）** 为非特权 Docker 环境启用较弱沙盒（降低安全性） |
| `sandbox.network.allowUnixSockets` | array | `[]` | **（仅限 macOS）** 沙盒中可访问的特定 Unix socket 路径。在 Linux 和 WSL2 上被忽略，因为 seccomp 过滤器无法检查 socket 路径；请改用 `allowAllUnixSockets` |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | 允许所有 Unix socket（覆盖 `allowUnixSockets`）。在 Linux 和 WSL2 上这是允许 Unix socket 的唯一方式，因为它跳过了原本会阻止 `socket(AF_UNIX, ...)` 调用的 seccomp 过滤器 |
| `sandbox.network.allowLocalBinding` | boolean | `false` | 允许绑定到 localhost 端口（macOS） |
| `sandbox.network.allowedDomains` | array | `[]` | 沙盒的网络域名允许列表 |
| `sandbox.network.deniedDomains` | array | `[]` | bash 沙盒的网络域名阻止列表。优先级高于 `allowedDomains` 中的通配符。支持 glob 模式（如 `"*.example.com"`）（v2.1.113） |
| `sandbox.network.httpProxyPort` | number | - | HTTP 代理端口 1-65535（自定义代理） |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 代理端口 1-65535（自定义代理） |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | 仅允许托管允许列表中的域名（托管设置） |
| `sandbox.network.allowMachLookup` | array | `[]` | （仅限 macOS）沙盒可查找的额外 XPC/Mach 服务名称。支持单个末尾 `*` 用于前缀匹配。需要通过 XPC 通信的工具（如 iOS 模拟器或 Playwright）需要此设置。示例：`["com.apple.coresimulator.*"]` |
| `sandbox.filesystem.allowWrite` | array | `[]` | 沙盒化命令可以写入的额外路径。数组在所有设置范围之间合并。还与 `Edit(...)` 允许权限规则的路径合并。前缀：`/`（绝对路径）、`~/`（主目录）、`./` 或无前缀（项目设置中为项目相对路径，用户设置中为 `~/.claude` 相对路径）。旧的 `//` 前缀用于绝对路径仍然有效。**注意：** 这与 [读/编辑权限规则](#tool-permission-syntax) 不同，后者使用 `//` 表示绝对路径，`/` 表示项目相对路径 |
| `sandbox.filesystem.denyWrite` | array | `[]` | 沙盒化命令不能写入的路径。数组在所有设置范围之间合并。还与 `Edit(...)` 拒绝权限规则的路径合并。与 `allowWrite` 相同的路径前缀约定 |
| `sandbox.filesystem.denyRead` | array | `[]` | 沙盒化命令不能读取的路径。数组在所有设置范围之间合并。还与 `Read(...)` 拒绝权限规则的路径合并。与 `allowWrite` 相同的路径前缀约定 |
| `sandbox.filesystem.allowRead` | array | `[]` | 在 `denyRead` 区域内重新允许读取访问的路径。优先级高于 `denyRead`。数组在所有设置范围之间合并。与 `allowWrite` 相同的路径前缀约定 |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **（仅限托管）** 仅尊重来自托管设置的 `allowRead` 路径。用户、项目和本地设置中的 `allowRead` 条目被忽略 |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | （仅限 macOS）允许访问系统 TLS 信任（`com.apple.trustd.agent`）；降低安全性 |
| `sandbox.bwrapPath` | string | - | **（仅限托管，Linux/WSL2）** bubblewrap（`bwrap`）二进制的绝对路径。覆盖自动 `PATH` 检测。仅在托管设置中生效，不在用户或项目设置中。示例：`/opt/admin/bwrap`（v2.1.133） |
| `sandbox.socatPath` | string | - | **（仅限托管，Linux/WSL2）** 用于沙盒网络代理的 `socat` 二进制的绝对路径。覆盖自动 `PATH` 检测。仅在托管设置中生效。示例：`/opt/admin/socat`（v2.1.133） |

**示例：**
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

| 键 | 类型 | 范围 | 描述 |
|-----|------|-------|-------------|
| `enabledPlugins` | object | 任意 | 启用/禁用特定插件 |
| `extraKnownMarketplaces` | object | 项目 | 添加自定义插件市场（通过 `.claude/settings.json` 团队共享） |
| `strictKnownMarketplaces` | array | 仅限托管 | 允许的市场列表 |
| `strictPluginOnlyCustomization` | boolean \| array | 仅限托管 | 阻止来自用户和项目来源的技能、agent、钩子和 MCP 服务器，使它们只能来自插件或托管设置。`true` 锁定所有四个层面；数组如 `["skills", "hooks"]` 仅锁定命名的层面 |
| `pluginSuggestionMarketplaces` | array | 仅限托管 | 允许列表，其中的市场插件可在会话期间作为上下文安装建议出现。限制哪些市场可以触发"你可能需要这个插件"提示（v2.1.152） |
| `skippedMarketplaces` | array | 任意 | 用户拒绝安装的市场 *（在 JSON schema 中，不在官方设置页面上）* |
| `skippedPlugins` | array | 任意 | 用户拒绝安装的插件 *（在 JSON schema 中，不在官方设置页面上）* |
| `pluginConfigs` | object | 任意 | 每插件 MCP 服务器配置（以 `plugin@marketplace` 为键）*（在 JSON schema 中，不在官方设置页面上）* |
| `blockedMarketplaces` | array | 仅限托管 | 阻止特定插件市场。每个条目可按源字符串、`hostPattern` 或 `pathPattern` 匹配 — 截至 v2.1.119，`hostPattern` 和 `pathPattern` 匹配器在任何下载触及文件系统前被正确执行，因此被阻止的市场永远不会到达磁盘 |
| `pluginTrustMessage` | string | 仅限托管 | 提示用户信任插件时显示的自定义消息 |

**市场源类型：** `github`、`git`、`directory`、`hostPattern`、`settings`、`url`、`npm`、`file`。使用 `source: 'settings'` 可在不设置托管市场仓库的情况下内联声明少量插件。

**示例：**
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
| `"sonnet"` | 最新 Sonnet 模型（Anthropic API 上为 Claude Sonnet 4.6；第三方提供商上为 4.5） |
| `"opus"` | 最新 Opus 模型（截至 v2.1.154，Anthropic API 上为 Claude Opus 4.8；Bedrock/Vertex/Foundry 上为 4.6）。自 v2.1.142 起也是快速模式默认。Opus 4.8 默认为 `high` 努力级别并支持 `/effort xhigh` |
| `"haiku"` | 快速 Haiku 模型 |
| `"sonnet[1m]"` | 带 1M token 上下文的 Sonnet |
| `"opus[1m]"` | 带 1M token 上下文的 Opus（自 v2.1.75 起为 Max、Team 和 Enterprise 的默认值） |
| `"opusplan"` | 规划时使用 Opus，执行时使用 Sonnet |
| `"fable"` | Claude Fable 5 — 长时推理模型。仅限 Anthropic API（v2.1.170+） |

**示例：**
```json
{
  "model": "opus"
}
```

> **注意（v2.1.144）：** `/model` 仅在**当前会话**中更改模型。在 `/model` 选择器中按 `d` 可将选择也设为默认。`model` 设置和 `ANTHROPIC_MODEL` 继续控制持久默认值。

### 模型覆盖

将 Anthropic 模型 ID 映射到 Bedrock、Vertex 或 Foundry 部署的特定提供商模型 ID。

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `effortLevel` | string | - | 跨会话持久化努力级别。接受 `"low"`、`"medium"`、`"high"` 或 `"xhigh"`（Opus 4.7 和 4.8，v2.1.111）。运行 `/effort low`、`/effort medium`、`/effort high` 或 `/effort xhigh` 时自动写入。支持 Opus 4.6、Sonnet 4.6、Opus 4.7 和 Opus 4.8（默认为 `high`）。不支持的级别会回退到活动模型支持的最高级别 |
| `fallbackModel` | array | - | 最多 3 个回退模型 ID，在主模型不可用时（如速率限制或容量问题）按顺序尝试。每个条目是模型 ID 或别名。Claude Code 首先尝试主模型；如果失败，按顺序尝试每个回退。在第一个成功响应时停止（v2.1.166） |
| `modelOverrides` | object | - | 将模型选择器条目映射到特定提供商 ID（如 Bedrock 推理配置文件 ARN）。每个键是模型选择器条目名称，每个值是提供商模型 ID |

**示例：**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### 努力级别

`/model` 命令提供了一个**努力级别**控制，用于调整模型每次响应时应用的推理量。在 `/model` UI 中使用 ← → 方向键循环切换努力级别。

| 努力级别 | 描述 |
|-------------|-------------|
| Max | 最大推理深度，仅 Opus 4.6 |
| XHigh | 扩展高推理深度，Opus 4.7 和 4.8（自 v2.1.111 起为 Opus 4.7 在所有计划上的默认值；在 Opus 4.8 上可用但默认为 `high`，v2.1.154） |
| High（Opus 4.6/Sonnet 4.6 默认） | 完整推理深度，最适合复杂任务 |
| Medium | 平衡推理，适合日常任务 |
| Low | 最小推理，最快响应 |

**使用方法：**
1. 运行 `/effort low`、`/effort medium` 或 `/effort high` 直接设置（v2.1.76+）
2. 或运行 `/model` → 选择模型 → 使用 **← →** 方向键调整
3. 设置通过 `settings.json` 中的 `effortLevel` 键持久化

**注意：** 努力级别可在 Max 和 Team 计划上用于 Opus 4.6、Sonnet 4.6、Opus 4.7 和 Opus 4.8。默认值在 v2.1.68 中从 High 改为 Medium，然后在 v2.1.94 中针对 API 密钥、Bedrock/Vertex/Foundry、Team 和 Enterprise 用户改回 **High**。在 v2.1.117 中，Pro/Max 订阅者在 Opus 4.6 和 Sonnet 4.6 上的默认值也从 `medium` 提高到 `high`，使所有层级统一为 `high`。v2.1.111 引入了 **`xhigh`**（当时仅限 Opus 4.7）并使其成为 Opus 4.7 在所有计划上的默认努力级别。**v2.1.154** 添加了 **Opus 4.8** 作为 Anthropic API 上的最新 Opus；它支持 `xhigh` 但默认为 `high`。截至 v2.1.75，1M 上下文窗口在 Max、Team 和 Enterprise 计划上默认可用于 Opus 4.6。

**努力级别环境变量传播：** 在技能文件内部，使用 `${CLAUDE_EFFORT}` 引用当前努力级别（v2.1.120）。截至 v2.1.133，相同的 `$CLAUDE_EFFORT` 变量也被注入到 Bash 工具子进程和钩子处理程序的环境中，因此 shell 脚本和钩子命令可以根据活跃的努力级别调整行为，无需读取单独的配置文件。

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

## 显示与用户体验

### 显示设置

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `statusLine` | object | - | 自定义状态行配置 |
| `outputStyle` | string | `"default"` | 输出样式（如 `"Explanatory"`） |
| `spinnerTipsEnabled` | boolean | `true` | 等待时显示提示 |
| `spinnerVerbs` | object | - | 自定义加载动画动词，包含 `mode`（"append" 或 "replace"）和 `verbs` 数组 |
| `spinnerTipsOverride` | object | - | 自定义加载动画提示，包含 `tips`（字符串数组）和可选的 `excludeDefault`（布尔值）。当 `excludeDefault` 为 `true` 时，仅显示自定义提示；为 `false` 或缺省时，自定义提示与内置提示合并。截至 v2.1.121，`excludeDefault: true` 也会抑制基于时间的加载动画提示 |
| `respectGitignore` | boolean | `true` | 在文件选择器中遵守 .gitignore |
| `prefersReducedMotion` | boolean | `false` | 减少 UI 中的动画和运动效果 |
| `syntaxHighlightingDisabled` | boolean | `false` | 在差异、代码块和文件预览中禁用语法高亮。与 `CLAUDE_CODE_SYNTAX_HIGHLIGHT` 环境变量不同，后者仅管理差异输出 |
| `fileSuggestion` | object | - | 自定义文件建议命令（见下方文件建议配置） |
| `autoScrollEnabled` | boolean | `true` | 在全屏模式下自动滚动对话。设为 `false` 以禁用自动滚动（v2.1.110）。v2.1.119 之前的版本将其存储在 `~/.claude.json` 中 |
| `editorMode` | string | `"normal"` | 输入提示的键绑定模式：`"normal"` 或 `"vim"`。在 `/config` 中显示为 **Editor mode**。v2.1.119 之前的版本将其存储在 `~/.claude.json` 中 |
| `showTurnDuration` | boolean | `true` | 在响应后显示轮次持续时间消息（如 "Cooked for 1m 6s"）。v2.1.119 之前的版本将其存储在 `~/.claude.json` 中 |
| `teammateMode` | string | `"auto"` | [agent 团队](https://code.claude.com/docs/en/agent-teams) 队友的显示方式：`"auto"`（在 tmux 或 iTerm2 中选择分割窗格，其他情况为进程内）、`"in-process"` 或 `"tmux"`。见 [选择显示模式](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode)。v2.1.119 之前的版本将其存储在 `~/.claude.json` 中 |
| `terminalProgressBarEnabled` | boolean | `true` | 在支持的终端中显示终端进度条（ConEmu、Ghostty 1.2.0+ 和 iTerm2 3.6.6+）。在 `/config` 中显示为 **Terminal progress bar**。v2.1.119 之前的版本将其存储在 `~/.claude.json` 中 |

| `preferredNotifChannel` | string | `"auto"` | 任务完成和权限提示通知的方式。值：`"auto"`、`"terminal_bell"`、`"iterm2"`、`"iterm2_with_bell"`、`"kitty"`、`"ghostty"`、`"notifications_disabled"`。默认 `"auto"` 在 iTerm2、Ghostty 和 Kitty 中发送桌面通知，在其他终端中不执行任何操作。设置 `"terminal_bell"` 可在任何终端中响铃字符。在 `/config` 中显示为 **Notifications**。见 [获取终端铃或通知](https://code.claude.com/docs/en/terminal-config#get-a-terminal-bell-or-notification) |

### 全局配置设置（`~/.claude.json`）

这些 IDE 相关偏好存储在 `~/.claude.json` 中，**而非** `settings.json`。

> **v2.1.119 迁移说明：** 截至 v2.1.119，`autoScrollEnabled`、`editorMode`、`showTurnDuration`、`teammateMode` 和 `terminalProgressBarEnabled` 已移入 `settings.json` 并在上方的显示设置表中有文档说明。早期版本将它们存储在此处。

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|
| `autoConnectIde` | boolean | `false` | 当 Claude Code 从外部终端启动时自动连接到正在运行的 IDE。在 VS Code 或 JetBrains 终端之外运行时，在 `/config` 中显示为 **Auto-connect to IDE (external terminal)** |
| `autoInstallIdeExtension` | boolean | `true` | 从 VS Code 终端运行时自动安装 Claude Code IDE 扩展。在 `/config` 中显示为 **Auto-install IDE extension**。也可通过 `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` 环境变量禁用 |
| `externalEditorContext` | boolean | `false` | 使用 `Ctrl+G` 打开外部编辑器时，将 Claude 之前的响应作为 `#` 注释上下文 prepend。设为 `true` 以启用 |
| `teammateDefaultModel` | string | `null` | [agent 团队](https://code.claude.com/docs/en/agent-teams) 队友在队长分派时的默认模型。`null` 继承队长的模型。在官方设置页面的"全局配置设置"下列出 |

### 工作区与团队

| 键 | 类型 | 描述 |
|-----|------|-------------|
| `sshConfigs` | object[] | SSH 连接定义，在 Desktop 中显示为下拉列表。每个条目必须包含 `id`、`name` 和 `sshHost`；可选 `sshPort`、`sshIdentityFile` 和 `startDirectory` |

**字段参考：**

| 字段 | 必需 | 描述 |
|-------|----------|-------------|
| `id` | 是 | SSH 连接条目的唯一标识符 |
| `name` | 是 | 在 Desktop 下拉列表中显示的名称 |
| `sshHost` | 是 | SSH 主机（如 `user@dev.example.com` 或 `dev.example.com`） |
| `sshPort` | 否 | SSH 端口号 |
| `sshIdentityFile` | 否 | SSH 身份文件（私钥）路径 |
| `startDirectory` | 否 | 连接后的初始工作目录 |

**示例：**
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

### 状态行配置

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

| 字段 | 描述 |
|-------|-------------|
| `type` | 设置为 `"command"` 以运行 shell 脚本 |
| `command` | 生成状态行输出的 shell 命令或脚本路径 |
| `padding` | 添加到状态行内容的额外水平间距（以字符为单位）。默认为 `0`。控制超出界面内置间距的相对缩进 |
| `refreshInterval` | 除事件驱动的更新外，每 N 秒重新运行命令。最小值为 `1`。适用于状态行显示基于时间的数据（如时钟）或后台子 agent 在主会话空闲时更改 git 状态的场景。不设置则仅在事件时运行（v2.1.97） |

**状态行输入字段：**

状态行命令通过 stdin 接收一个 JSON 对象。完整的 JSON schema 和示例见 [状态行文档](https://code.claude.com/docs/en/statusline)。

| 字段 | 描述 |
|-------|-------------|
| `model.id`、`model.display_name` | 当前模型标识符和显示名称 |
| `cwd`、`workspace.current_dir` | 当前工作目录（两者包含相同的值；首选 `workspace.current_dir`） |
| `workspace.project_dir` | Claude Code 启动的目录（如果工作目录发生变化，可能与 `cwd` 不同） |
| `workspace.added_dirs` | 通过 `/add-dir` 或 `--add-dir` 添加的额外目录 |
| `workspace.git_worktree` | 在通过 `git worktree add` 创建的链接工作树内部时的 git worktree 名称。在主工作树中不存在（v2.1.97） |
| `cost.total_cost_usd` | 会话总成本（美元） |
| `cost.total_duration_ms` | 自会话启动以来的总墙钟时间（毫秒） |
| `cost.total_api_duration_ms` | 等待 API 响应的总时间（毫秒） |
| `cost.total_lines_added`、`cost.total_lines_removed` | 会话期间更改的代码行数 |
| `context_window.total_input_tokens`、`context_window.total_output_tokens` | 跨会话的累积 token 计数 |
| `context_window.context_window_size` | 最大上下文窗口大小（token），默认 200000，扩展上下文 1000000 |
| `context_window.used_percentage` | 预计算的已使用上下文窗口百分比 |
| `context_window.remaining_percentage` | 预计算的剩余上下文窗口百分比 |
| `context_window.current_usage` | 最后一次 API 调用的 token 计数（输入、输出、缓存 token） |
| `exceeds_200k_tokens` | 最近一次 API 响应的总 token 是否超过 200k（固定阈值） |
| `rate_limits.five_hour.used_percentage` | 五小时速率限制使用百分比（v2.1.80+） |
| `rate_limits.five_hour.resets_at` | 五小时速率限制重置时间戳（Unix 纪元秒） |
| `rate_limits.seven_day.used_percentage` | 七天速率限制使用百分比 |
| `rate_limits.seven_day.resets_at` | 七天速率限制重置时间戳（Unix 纪元秒） |
| `session_id` | 唯一会话标识符 |
| `session_name` | 使用 `--name` 或 `/rename` 设置的自定义会话名称。未设置自定义名称时不存在 |
| `transcript_path` | 对话记录文件路径 |
| `version` | Claude Code 版本 |
| `output_style.name` | 当前输出样式名称 |
| `vim.mode` | 启用 vim 模式时的当前 vim 模式（`NORMAL` 或 `INSERT`） |
| `agent.name` | 使用 `--agent` 标志或 agent 设置运行时的 agent 名称 |
| `effort.level` | 当前推理努力级别（`low`、`medium`、`high`、`xhigh` 或 `max`）。反映实时会话值，包括会话中的 `/effort` 更改。当前模型不支持努力级别参数时不存在（v2.1.121） |
| `thinking.enabled` | 是否为会话启用扩展思考（v2.1.121） |
| `worktree.name` | 活跃 worktree 的名称（仅在 `--worktree` 会话中出现） |
| `worktree.path` | worktree 目录的绝对路径 |
| `worktree.branch` | worktree 的 git 分支名称。基于钩子的 worktree 不存在 |
| `worktree.original_cwd` | 进入 worktree 之前的目录 |
| `worktree.original_branch` | 进入 worktree 之前检出的 git 分支。基于钩子的 worktree 不存在 |
| `github` | 检测到分支时当前分支的 GitHub 仓库和拉取请求信息 — 仓库标识和关联的 PR（v2.1.145） |

### 文件建议配置

文件建议脚本通过 stdin 接收一个 JSON 对象（如 `{"query": "src/comp"}`），必须输出最多 15 个文件路径（每行一个）。

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**示例：**
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

| 键 | 类型 | 描述 |
|-----|------|-------------|
| `awsAuthRefresh` | string | 刷新 AWS 认证的脚本（修改 `.aws` 目录） |
| `awsCredentialExport` | string | 输出包含 AWS 凭据的 JSON 的脚本 |

**示例：**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| 键 | 类型 | 描述 |
|-----|------|-------------|
| `otelHeadersHelper` | string | 生成动态 OpenTelemetry 请求头的脚本 |

**示例：**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## 环境变量（通过 `env`）

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

| 变量 | 描述 |
|----------|-------------|
| `ANTHROPIC_API_KEY` | 用于认证的 API 密钥 |
| `ANTHROPIC_AUTH_TOKEN` | OAuth token |
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude.ai 认证的 OAuth 访问 token。SDK 和自动化环境中 `/login` 的替代方案。优先级高于钥匙串存储的凭据 |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | Claude.ai 认证的 OAuth 刷新 token。设置后，`claude auth login` 会直接使用此 token 交换，而非打开浏览器。需要 `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | 刷新 token 被授予的以空格分隔的 OAuth 作用域（如 `"user:profile user:inference user:sessions:claude_code"`）。设置 `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` 时需要 |
| `ANTHROPIC_WORKSPACE_ID` | [工作负载身份联合](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation)的工作空间 ID。当你的联合规则作用域超过一个工作空间时设置，以便 token 交换知道目标工作空间（v2.1.141） |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `ANTHROPIC_BEDROCK_BASE_URL` | 覆盖 Bedrock 端点 URL |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | 覆盖 Bedrock Mantle 端点 URL。见 [Mantle 端点](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock 服务层级：`default`、`flex` 或 `priority`。作为 `X-Amzn-Bedrock-Service-Tier` 请求头在每个请求上发送。见 [Amazon Bedrock 服务层级](https://code.claude.com/docs/en/amazon-bedrock#service-tiers)（v2.1.122） |
| `ANTHROPIC_AWS_API_KEY` | AWS 上 Claude Platform 的工作空间 API 密钥 |
| `ANTHROPIC_AWS_BASE_URL` | 覆盖 AWS 上 Claude Platform 的端点 URL |
| `ANTHROPIC_AWS_WORKSPACE_ID` | AWS 上 Claude Platform 所需的工作空间 ID |
| `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` | 由嵌入 Claude Code 并代表用户管理模型提供商路由的宿主平台设置。设置后，`settings.json` 中的提供商选择/端点/认证环境变量（如 `CLAUDE_CODE_USE_BEDROCK`、`ANTHROPIC_BASE_URL`、`ANTHROPIC_API_KEY`）被忽略，因此用户设置无法覆盖宿主的路由。Bedrock/Vertex/Foundry 的自动遥测选择退出也会被跳过，因此遥测遵循标准的 `DISABLE_TELEMETRY` 选择退出（v2.1.126） |
| `ANTHROPIC_VERTEX_BASE_URL` | 覆盖 Vertex AI 端点 URL |
| `ANTHROPIC_BETAS` | 以逗号分隔的 Anthropic beta 请求头值 |
| `ANTHROPIC_VERTEX_PROJECT_ID` | Vertex AI 的 GCP 项目 ID |
| `GCLOUD_PROJECT` | Vertex AI 请求的 GCP 项目 ID（覆盖 `ANTHROPIC_VERTEX_PROJECT_ID`） |
| `GOOGLE_APPLICATION_CREDENTIALS` | 用于 Vertex AI 认证的 GCP 服务账户凭据文件路径 |
| `GOOGLE_CLOUD_PROJECT` | Vertex AI 请求的 GCP 项目 ID（覆盖 `ANTHROPIC_VERTEX_PROJECT_ID`） |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | 要作为自定义条目添加到 `/model` 选择器中的模型 ID。用于使非标准或网关特定的模型可选择，而不替换内置别名 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | `/model` 选择器中自定义模型条目的显示名称。未设置时默认为模型 ID |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | `/model` 选择器中自定义模型条目的显示描述。未设置时默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES` | 覆盖自定义模型条目的能力检测。以逗号分隔的值（如 `effort,thinking`）。当自定义模型支持自动检测无法确认的功能时需要。见 [模型配置](https://code.claude.com/docs/en/model-config#customize-pinned-model-display-and-capabilities) |
| `ANTHROPIC_MODEL` | 要使用的模型名称。接受别名（`sonnet`、`opus`、`haiku`）或完整模型 ID。覆盖 `model` 设置 |
| `INIT_PROMPT` | 会话初始化时注入的自定义系统提示 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 使用自定义模型 ID 覆盖 Haiku 模型别名（如用于第三方部署） |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上使用固定模型时，自定义 `/model` 选择器中 Haiku 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Haiku 条目的描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | 覆盖固定 Haiku 模型的能力检测。以逗号分隔的值（如 `effort,thinking`）。当固定模型支持自动检测无法确认的功能时需要 |
| `CLAUDECODE` | 在 Claude Code 生成的 shell 环境（Bash 工具、tmux 会话）中设置为 `1`。不在钩子或状态行命令中设置。用于检测脚本是否在 Claude Code shell 内部运行 |
| `CLAUDE_CODE_SESSION_ID` | 只读。在 Bash 和 PowerShell 工具子进程中自动设置为当前会话 ID。与传递给钩子的 `session_id` 字段匹配。在 `/clear` 时更新。用于将脚本和外部工具与启动它们的 Claude Code 会话关联（v2.1.132）。在 `--resume` 时也会注入到 stdio MCP 服务器环境中（v2.1.163 变更日志）*（在 v2.1.163 变更日志中；尚未在官方环境变量页面 — 只读）* |
| `AI_AGENT` | 由 Claude Code 在子进程环境（Bash 工具、钩子、MCP stdio 服务器）中自动设置。通用标志，标识父进程为 AI agent — 适用于在任何 AI agent（而非检查每个 agent 特定变量如 `CLAUDECODE`）调用时调整行为的工具 *（在 v2.1.120 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | 设置为 `1` 以在组织状态检查因网络错误失败时允许快速模式。当企业代理阻止状态端点时有用 |
| `CLAUDE_CODE_USE_BEDROCK` | 使用 AWS Bedrock（`1` 启用） |
| `CLAUDE_CODE_USE_VERTEX` | 使用 Google Vertex AI（`1` 启用） |
| `CLAUDE_CODE_USE_FOUNDRY` | 使用 Microsoft Foundry（`1` 启用） |
| `CLAUDE_CODE_USE_MANTLE` | 使用 Bedrock [Mantle 端点](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint)（`1` 启用） |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | 设置为 `1` 以在 Windows 上启用 PowerShell 工具（选择加入预览）。启用后，Claude 可以原生运行 PowerShell 命令，而非通过 Git Bash 路由。仅在原生 Windows 上支持，不在 WSL 上（v2.1.84） |
| `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` | 设置为 `1` 以停止 Claude Code 在生成 PowerShell 用于工具调用、钩子和状态行命令时传递 `-ExecutionPolicy Bypass`，改为尊重机器的有效执行策略。默认情况下，Claude Code 在进程作用域绕过执行策略，以便 `.ps1` 脚本和模块导入在默认 Restricted 的 Windows 上工作。永远不会覆盖组策略 `MachinePolicy`/`UserPolicy`（v2.1.143） |
| `CLAUDE_CODE_REMOTE` | 只读。当 Claude Code 作为云会话运行时自动设置为 `true`。从钩子或设置脚本中读取以检测是否在云环境中 |
| `CLAUDE_CODE_REMOTE_SESSION_ID` | 只读。在云会话中自动设置为当前会话的 ID。读取此值以构建返回会话记录的链接 |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | 自动生成的远程控制会话名称前缀。默认为机器主机名 |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 启用/禁用遥测（`0` 或 `1`） |
| `DISABLE_ERROR_REPORTING` | 禁用错误报告（`1` 禁用） |
| `DISABLE_AUTOUPDATER` | 设置为 `1` 以禁用针对 npm 注册表的自动更新检查。也可配置为仅启动变量 — 见 [CLI 启动标志](./claude-cli-startup-flags.md#environment-variables) |
| `DISABLE_UPDATES` | 设置为 `1` 以完全阻止所有更新路径 — 自动检查、通知和手动 `claude update`。比 `DISABLE_AUTOUPDATER` 更严格，后者仅禁用后台检查。用于必须阻止所有更新直到显式重新启用的环境 *（在 v2.1.118 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | 设置为 `1` 以让 Claude Code 在新版本可用时在后台运行你的包管理器的升级命令。适用于 Homebrew 和 WinGet 安装。其他包管理器继续显示升级命令而不执行。见 [自动更新](https://code.claude.com/docs/en/setup#auto-updates)（v2.1.129） |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | 设置为 `1` 以在 `ANTHROPIC_BASE_URL` 指向 Anthropic 兼容网关（如 LiteLLM、Kong 或内部代理）时从网关的 `/v1/models` 端点填充 `/model` 选择器。默认关闭，因为由共享 API 密钥支持的网关否则会暴露密钥可访问的每个模型。发现的模型仍受 `availableModels` 允许列表过滤（v2.1.129，从先前自动发现的选择加入更改） |
| `DISABLE_TELEMETRY` | 禁用遥测（`1` 禁用） |
| `DO_NOT_TRACK` | 标准选择退出变量；设置为 `1` 以选择不收集遥测。被 `DISABLE_TELEMETRY` 尊重 |
| `MCP_TIMEOUT` | MCP 启动超时（毫秒） |
| `CLAUDE_CODE_MCP_ALLOWLIST_ENV` | 仅使用安全基线环境生成 stdio MCP 服务器，剥离大多数继承的环境变量以防止凭据泄露到不受信任的服务器进程中 |
| `MAX_MCP_OUTPUT_TOKENS` | 最大 MCP 输出 token（默认：25000）。输出超过 10,000 token 时显示警告 |
| `API_TIMEOUT_MS` | API 请求超时（毫秒，默认：600000） |
| `BASH_MAX_TIMEOUT_MS` | Bash 命令超时 |
| `BASH_MAX_OUTPUT_LENGTH` | 最大 bash 输出长度 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 自动压缩阈值百分比（1-100）。默认约 95%。设置较低值（如 `50`）以更早触发压缩。高于 95% 的值无效。使用 `/context` 监控当前使用情况。示例：`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | 覆盖 Claude Code 假设的活跃模型的上下文窗口大小。仅在同时设置 `DISABLE_COMPACT` 时生效。当通过 `ANTHROPIC_BASE_URL` 路由到上下文窗口与其名称内置大小不匹配的模型时使用 |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 在 bash 调用之间保持 cwd（`1` 启用） |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 禁用后台任务（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | 设置为 `1` 以关闭后台 agent 和 agent 视图（`claude agents`、`--bg`、`/background`、按需监督器）。环境变量等效于 `disableAgentView` 设置 *（在官方设置页面上引用；未列在环境变量页面上）* |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | 启用实验性 agent 团队功能（`1` 启用）。允许在会话中生成协调的子 agent 团队 |
| `CLAUDE_CODE_DISABLE_WORKFLOWS` | 设置为 `1` 以禁用 [动态工作流](https://code.claude.com/docs/en/workflows)（`/workflows`）和内置的工作流斜杠命令。环境变量等效于 `disableWorkflows` 设置 |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` | 设置为 `1` 以隐藏 Claude Code 的内置能力（bundled skills）。`disableBundledSkills` 设置的环境变量等效 *(在 v2.1.170 变更日志中，尚未在官方环境变量页面)* |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | 设置为 `1` 以在 Amazon Bedrock、Google Cloud Vertex AI 和 Microsoft Foundry 上启用 [自动模式](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)。对 Anthropic API 无效，因为自动模式默认可用（v2.1.158） |
| `ENABLE_TOOL_SEARCH` | MCP 工具搜索阈值（如 `auto:5`） |
| `ENABLE_PROMPT_CACHING_1H` | 选择加入 1 小时提示缓存 TTL。替换已弃用的 `ENABLE_PROMPT_CACHING_1H_BEDROCK` *（在 v2.1.108 变更日志中，尚未在官方环境变量页面）* |
| `FORCE_PROMPT_CACHING_5M` | 强制 5 分钟提示缓存 TTL *（在 v2.1.108 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | 选择退出离开摘要/空闲会话回顾。设置为 `0` 以禁用。与 `awaySummaryEnabled` 设置配合使用（v2.1.110） |
| `DISABLE_PROMPT_CACHING` | 禁用所有提示缓存（`1` 禁用） |
| `DISABLE_PROMPT_CACHING_HAIKU` | 禁用 Haiku 提示缓存 |
| `DISABLE_PROMPT_CACHING_SONNET` | 禁用 Sonnet 提示缓存 |
| `DISABLE_PROMPT_CACHING_OPUS` | 禁用 Opus 提示缓存 |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | 在 Bedrock 上请求 1 小时缓存 TTL（`1` 启用）*（不在官方文档中 — 未经验证；v2.1.108 变更日志称已弃用，被 `ENABLE_PROMPT_CACHING_1H` 替换）* |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | 禁用实验性 beta 功能（`1` 禁用） |
| `CLAUDE_CODE_SHELL` | 覆盖自动 shell 检测 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | 覆盖默认文件读取 token 限制 |
| `CLAUDE_CODE_GLOB_HIDDEN` | 设置为 `false` 以在 Claude 调用 Glob 工具时从结果中排除点文件。默认包含。不影响 `@` 文件自动补全、`ls`、Grep 或 Read |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | 设置为 `false` 以使 Glob 工具遵守 `.gitignore` 模式。默认情况下，Glob 返回所有匹配文件，包括被 gitignore 的文件。不影响 `@` 文件自动补全，它有自己的 `respectGitignore` 设置 |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Glob 文件发现的超时时间（秒） |
| `CLAUDE_CODE_ENABLE_TASKS` | 控制会话是否使用结构化 Task 工具（`TaskCreate`、`TaskUpdate`、`TaskGet`、`TaskList`）或旧的 `TodoWrite` 工具。截至 v2.1.142，Task 工具在所有模式下为默认。设置为 `0` 以回退到 `TodoWrite` |
| `CLAUDE_CODE_SIMPLE` | 设置为 `1` 以使用最小系统提示和仅 Bash、文件读取和文件编辑工具运行。也可配置为仅启动变量 — 见 [CLI 启动标志](./claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | 空闲持续时间后自动退出 SDK 模式（毫秒） |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | 禁用自适应思考（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_THINKING` | 强制禁用扩展思考（`1` 禁用） |
| `DISABLE_INTERLEAVED_THINKING` | 阻止发送交错思考 beta 请求头（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | 禁用 1M token 上下文窗口（`1` 禁用） |
| `CLAUDE_CODE_ACCOUNT_UUID` | 覆盖认证用账户 UUID |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | 禁用与 git 相关的系统提示指令 |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | 设置为 `0` 以从系统提示中省略 Claude Code 归属块 |
| `CLAUDE_CODE_NEW_INIT` | 设置为 `true` 使 `/init` 运行交互式设置流程。在探索代码库之前询问要生成哪些文件（CLAUDE.md、技能、钩子）。不设置时，`/init` 会自动生成 CLAUDE.md |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | 一个或多个只读插件种子目录的路径，Unix 上以 `:` 分隔，Windows 上以 `;` 分隔。将预填充的插件捆绑到容器镜像中。Claude Code 在启动时从这些目录注册市场并使用预缓存的插件而无需重新克隆 |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | 启用 Claude.ai MCP 服务器 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 设置努力级别：`low`、`medium`、`high`、`xhigh`（仅 Opus 4.7，v2.1.111）、`max`（仅 Opus 4.6）或 `auto`（使用模型默认）。优先级高于 `/effort` 和 `effortLevel` 设置。也可配置为仅启动变量 — 见 [CLI 启动标志](./claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_EFFORT` | 只读。注入到 Bash 工具子进程和钩子处理程序中，带有活跃的努力级别，以便 shell 脚本和钩子可以根据当前层级调整行为（`CLAUDE_CODE_EFFORT_LEVEL` 的配套；v2.1.133）。在技能文件内部使用 `${CLAUDE_EFFORT}` *（在变更日志中，不在官方环境变量页面 — 只读，不可用户配置）* |
| `CLAUDE_CODE_ALWAYS_ENABLE_EFFORT` | 设置为 `1` 以在所有模型上强制启用努力参数，即使通常不支持努力级别选择的模型。允许 `/effort` 和 `effortLevel` 设置在标准努力能力集合之外的模型上生效（v2.1.154）*（在 v2.1.154 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_MAX_TURNS` | 停止前的最大 agentic 轮次 *（不在官方文档中 — 未经验证）* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 等效于设置 `DISABLE_AUTOUPDATER`、`DISABLE_FEEDBACK_COMMAND`、`DISABLE_ERROR_REPORTING` 和 `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | 跳过首次运行设置设置流程 *（不在官方文档中 — 未经验证）* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | 覆盖提示缓存行为 *（不在官方文档中 — 未经验证）* |
| `CLAUDE_CODE_DISABLE_TOOLS` | 以逗号分隔的要禁用的工具列表 *（不在官方文档中 — 未经验证）* |
| `CLAUDE_CODE_DISABLE_MCP` | 禁用所有 MCP 服务器（`1` 禁用）*（不在官方文档中 — 未经验证）* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | 每次响应的最大输出 token。默认：32,000（截至 v2.1.77，Opus 4.6 为 64,000）。上限：64,000（截至 v2.1.77，Opus 4.6 和 Sonnet 4.6 为 128,000） |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | 完全禁用快速模式（`1` 禁用） |
| `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` | **已在 v2.1.160 中移除** — 环境变量现在是空操作。快速模式在默认模型上运行，不受此变量影响。之前将 [快速模式](https://code.claude.com/docs/en/fast-mode) 固定到 Claude Opus 4.6 而非默认值（v2.1.142–v2.1.159） |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | 设置为 `1` 以禁用流请求中途失败时的非流回退。流错误会传播到重试层。当代理或网关导致回退产生重复工具执行时有用（v2.1.83） |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | 中止停滞的流（`1` 启用） |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | 在 Anthropic API 上默认启用（v2.1.139+）；设置为 `0` 以选择退出 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 禁用自动 memory（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | 禁用 `/rewind` 的文件检查点（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | 禁用附件处理（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | 阻止加载 CLAUDE.md 文件（`1` 禁用） |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | 从启动时通过 `--add-dir` 指定的额外目录加载 CLAUDE.md memory 文件（`1` 启用）。也可配置为仅启动变量 — 见 [CLI 启动标志](./claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS` | 跳过从系统范围的托管技能目录加载技能（`1` 禁用） |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | 如果上次会话在轮次中间结束则自动恢复（`1` 启用） |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | 设置为 `1` 以跳过将提示历史和会话记录写入磁盘。设置此变量的会话不会出现在 `--resume`、`--continue` 或上箭头历史中。适用于临时脚本会话 |
| `CLAUDE_CODE_USER_EMAIL` | 同步提供用户邮箱以进行认证 |
| `CLAUDE_CODE_ORGANIZATION_UUID` | 同步提供组织 UUID 以进行认证 |
| `CLAUDE_CONFIG_DIR` | 自定义配置目录（覆盖默认 `~/.claude`） |
| `CLAUDE_CODE_TMPDIR` | 覆盖用于内部临时文件的临时目录。Claude Code 将 `/claude/` 追加到此路径。默认：Unix/macOS 上为 `/tmp`，Windows 上为 `os.tmpdir()` |
| `ANTHROPIC_CUSTOM_HEADERS` | API 请求的自定义请求头（`Name: Value` 格式，多个请求头以换行符分隔） |
| `CLAUDE_CODE_EXTRA_BODY` | 合并到每个 API 请求体顶层的 JSON 对象。用于注入特定于供应商的字段（如自定义网关的路由提示） |
| `CLAUDE_CODE_PROPAGATE_TRACEPARENT` | 设置为 `1` 以在通过自定义代理路由时将 W3C `traceparent` 请求头传播到请求中，将 Claude Code 追踪链接到你的上游遥测 |
| `ANTHROPIC_FOUNDRY_API_KEY` | 用于 Microsoft Foundry 认证的 API 密钥 |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry 资源的基准 URL |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Foundry 资源名称 |
| `AWS_BEARER_TOKEN_BEDROCK` | 用于认证的 Bedrock API 密钥 |
| `ANTHROPIC_SMALL_FAST_MODEL` | **已弃用** — 请改用 `ANTHROPIC_DEFAULT_HAIKU_MODEL` |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | 已弃用的 Haiku 类模型覆盖的 AWS 区域 |
| `CLAUDE_CODE_SHELL_PREFIX` | 预加到 bash 命令的命令前缀 |
| `BASH_DEFAULT_TIMEOUT_MS` | 默认 bash 命令超时（毫秒） |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | 跳过 Bedrock 的 AWS 认证（`1` 跳过） |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | 跳过 Foundry 的 Azure 认证（`1` 跳过） |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | 跳过 Bedrock Mantle 的 AWS 认证（如使用 LLM 网关时） |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | 跳过 Vertex 的 Google 认证（`1` 跳过） |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | 允许代理执行 DNS 解析 |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper` 的凭据刷新间隔（毫秒） |
| `CLAUDE_CODE_CLIENT_CERT` | 用于 mTLS 的客户端证书路径 |
| `CLAUDE_CODE_CLIENT_KEY` | 用于 mTLS 的客户端私钥路径 |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | 加密 mTLS 密钥的密码短语 |
| `CLAUDE_CODE_CERT_STORE` | TLS 连接的 CA 证书来源的逗号分隔列表：`bundled`（Claude Code 附带的 Mozilla CA 集）和/或 `system`（操作系统信任存储）。默认：`bundled,system`。本机二进制发行版需要系统集成；在 Node.js 运行时中，无论此值如何，仅使用内置集合（v2.1.101） |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | 插件市场 git 克隆超时（毫秒，默认：120000） |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | 设置为 `1` 以通过 HTTPS 而非 SSH 克隆 GitHub `owner/repo` 简写插件源。适用于插件安装/更新和 `/plugin marketplace add`/`update`。适用于没有为 `github.com` 配置 SSH 密钥的 CI 运行器或容器（v2.1.141） |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | 覆盖插件根目录 |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | 跳过自动添加官方市场（`1` 禁用） |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | 在首次查询前等待插件安装完成（`1` 启用） |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | 同步插件安装的超时（毫秒） |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | 设置为 `1` 以在 `git pull` 失败时保留现有的市场缓存，而非擦除并重新克隆。适用于离线或气隙环境中重新克隆会同样失败的场景 |
| `CLAUDE_CODE_ENABLE_BACKGROUND_PLUGIN_REFRESH` | 在后台安装完成后在会话轮次边界刷新插件状态（`1` 启用）。不设置时，新安装的插件在下一个会话生效 |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | 从 UI 隐藏邮箱/组织信息 *（不在官方文档中 — 未经验证）* |
| `CLAUDE_CODE_DISABLE_CRON` | 禁用计划/cron 任务（`1` 禁用） |
| `DISABLE_INSTALLATION_CHECKS` | 禁用安装警告 |
| `DISABLE_FEEDBACK_COMMAND` | 禁用 `/feedback` 命令。旧名称 `DISABLE_BUG_COMMAND` 也被接受 |
| `DISABLE_DOCTOR_COMMAND` | 隐藏 `/doctor` 命令（`1` 禁用） |
| `DISABLE_LOGIN_COMMAND` | 隐藏 `/login` 命令（`1` 禁用） |
| `DISABLE_LOGOUT_COMMAND` | 隐藏 `/logout` 命令（`1` 禁用） |
| `DISABLE_UPGRADE_COMMAND` | 隐藏 `/upgrade` 命令（`1` 禁用） |
| `DISABLE_EXTRA_USAGE_COMMAND` | 隐藏 `/extra-usage` 命令 — 在 v2.1.144 中重命名为 `/usage-credits`，尽管此环境变量名称不变（`1` 禁用） |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | 隐藏 `/install-github-app` 命令（`1` 禁用） |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | 禁用装饰文本和非必要模型调用 *（不在官方文档中 — 未经验证）* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | 覆盖调试日志文件目录路径 |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | 最低调试日志级别 |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | 强制自动后台化长时间任务（`1` 启用） |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | 阻止将 Opus 4.0/4.1 重新映射到更新的模型（`1` 禁用） |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | 触发所有主模型的回退模型，而非仅默认模型（`1` 启用） |
| `CCR_FORCE_BUNDLE` | 设置为 `1` 以强制 `claude --remote` 捆绑并上传本地仓库，即使 GitHub 访问可用。也可配置为仅启动变量 — 见 [CLI 启动标志](./claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_GIT_BASH_PATH` | 仅限 Windows：Git Bash 可执行文件（`bash.exe`）的路径。当 Git Bash 已安装但不在 PATH 中时使用 |
| `DISABLE_COST_WARNINGS` | 禁用成本警告消息 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 覆盖子 agent 的模型（如 `haiku`、`sonnet`） |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | 设置为 `1` 以从子进程环境（Bash 工具、钩子、MCP stdio 服务器）中剥离 Anthropic 和云提供商凭据。用于深度防御，当子进程不应继承 API 密钥时（v2.1.83） |
| `CLAUDE_CODE_SCRIPT_CAPS` | JSON 对象，限制在设置 `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` 时每会话可调用特定脚本的次数。键是匹配命令文本的子字符串；值是整数调用限制。例如，`{"deploy.sh": 2}` 允许 `deploy.sh` 最多调用两次。匹配是基于子字符串的；不检测通过 `xargs` 或 `find -exec` 的运行时扇出 — 这是深度防御控制 |
| `CLAUDE_CODE_PERFORCE_MODE` | 设置为 `1` 以启用 Perforce 感知写入保护。设置后，如果目标文件缺少所有者写入位（Perforce 在同步文件上清除直到 `p4 edit` 打开它们），Edit、Write 和 NotebookEdit 会以 `p4 edit <file>` 提示失败。防止 Claude Code 绕过 Perforce 更改跟踪（v2.1.98） |
| `CLAUDE_CODE_MAX_RETRIES` | 覆盖 API 请求重试次数（默认：10） |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | 最大并行只读工具数（默认：10） |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | 在 SDK 模式下禁用内置子 agent 类型（`1` 禁用） |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | 在 SDK 模式下跳过 MCP 工具的 `mcp__<server>__` 前缀（`1` 启用） |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` | 后台子 agent 的停滞超时（毫秒，默认：600000 / 10 分钟）。如果子 agent 在此持续时间内无输出则被终止 |
| `MCP_CONNECTION_NONBLOCKING` | 在 `-p` 模式下设置为 `true` 以完全跳过 MCP 连接等待。将 `--mcp-config` 服务器连接限制在 5 秒，而非阻塞在最慢的服务器上 *（在 v2.1.89 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd 钩子超时（毫秒，替换硬编码的 1.5 秒限制） |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | 禁用反馈调查提示（`1` 禁用） |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | 设置为 `1` 以在 Anthropic 非必要流量被阻止时将会话质量调查路由到你自己的 OpenTelemetry 收集器。调查评分仅作为 OTEL 事件发送到已配置的收集器 — 不向 Anthropic 发送调查数据。当设置 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`、`DISABLE_TELEMETRY` 或 `DO_NOT_TRACK` 时适用；否则无效。`CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` 和组织产品反馈策略优先级更高（v2.1.136） |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | 禁用终端标题更新（`1` 禁用） |
| `CLAUDE_CODE_TMUX_TRUECOLOR` | 设置为 `1` 以允许 tmux 内的 24 位真彩色输出。默认情况下，Claude Code 在设置 `$TMUX` 时将颜色限制为 256 色，因为 tmux 不会传递真彩色转义序列除非配置。在将 `set -ga terminal-overrides ',*:Tc'` 添加到 `~/.tmux.conf` 后设置此变量 |
| `CLAUDE_CODE_NO_FLICKER` | 设置为 `1` 以启用无闪烁备用屏幕渲染。消除全屏重绘期间的视觉闪烁（v2.1.88） |
| `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT` | 设置为 `1` 以在全屏渲染中每帧重绘整个屏幕。当部分重绘在不常见的终端模拟器中产生视觉伪影时使用 |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | 设置为 `1` 以禁用全屏渲染并使用经典的主屏幕渲染器。对话保留在终端的原生滚动缓冲区中，因此 `Cmd+f` 和 tmux 复制模式照常工作。优先级高于 `CLAUDE_CODE_NO_FLICKER` 和 `tui` 设置。你也可以通过 `/tui default` 切换（v2.1.132） |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | 设置为 `1` 以在终端支持但未自动检测时强制启用 DEC 私有模式 2026 同步输出。适用于实现 BSU/ESU 但不回复能力探测的模拟器，如 Emacs `eat`。在 tmux 下无效（v2.1.129） |
| `CLAUDE_CODE_SCROLL_SPEED` | 全屏渲染的鼠标滚轮滚动倍率。增加以更快滚动，减少以更精细控制 |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | 设置为 `1` 以禁用全屏渲染中的虚拟滚动并渲染记录中的每条消息。如果在全屏模式下滚动显示消息应出现的空白区域时使用 |
| `CLAUDE_CODE_DISABLE_MOUSE` | 设置为 `1` 以禁用全屏渲染中的鼠标跟踪。当鼠标事件干扰终端多路复用器或辅助工具时有用 |
| `CLAUDE_CODE_HIDE_CWD` | 设置为 `1` 以在 Claude Code 启动标志横幅中隐藏当前工作目录。适用于屏幕录制、演示或共享会话，其中 CWD 路径会泄露有关主机或项目布局的信息（v2.1.119） |
| `CLAUDE_CODE_ACCESSIBILITY` | 设置为 `1` 以保持原生终端光标可见，用于屏幕阅读器和辅助工具 |
| `CLAUDE_CODE_NATIVE_CURSOR` | 设置为 `1` 以在输入光标位置显示终端自己的光标，而非 Claude Code 的自定义光标字符 |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | 设置为 `0` 以在差异输出中禁用语法高亮 |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | 跳过自动 IDE 扩展安装（`1` 跳过） |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | 覆盖自动 IDE 连接行为 |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | 覆盖 IDE 主机地址用于连接 |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | 跳过 IDE lockfile 验证（`1` 跳过） |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | OTel 请求头助手脚本的去抖动间隔（毫秒） |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | OpenTelemetry flush 超时（毫秒） |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | OpenTelemetry 关闭超时（毫秒） |
| `CLAUDE_ENABLE_BYTE_WATCHDOG` | 设置为 `1` 以强制启用字节级流空闲看门狗，或 `0` 以强制禁用。未设置时，看门狗默认为 Anthropic API 连接启用。当 `CLAUDE_STREAM_IDLE_TIMEOUT_MS` 设置的持续时间内线上无字节到达时，字节看门狗中止连接（最小 5 分钟），独立于事件级看门狗 |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | 流空闲看门狗的超时（毫秒）。应用两个看门狗：**字节级**（默认和最小 `300000` / 5 分钟，当线上无字节到达时中止）和**事件级**（默认 `90000` / 90 秒，无最小值，当无 SSE 事件到达时中止）。字节看门狗默认为 Anthropic API 连接启用；通过 `CLAUDE_ENABLE_BYTE_WATCHDOG` 控制。如果长时间运行的工具或慢速网络导致过早超时错误，请增加事件超时 |
| `OTEL_LOG_TOOL_DETAILS` | 设置为 `1` 以在 OpenTelemetry 事件中包含 `tool_parameters`。出于隐私原因默认省略 *（在 v2.1.85 变更日志中，尚未在官方环境变量页面）* |
| `OTEL_LOG_RAW_API_BODIES` | 设置为 `1` 以将完整的 API 请求和响应体作为 OpenTelemetry 日志事件发出。出于隐私和负载大小原因默认省略。适用于在网关或代理上调试 *（在 v2.1.111 变更日志中，尚未在官方环境变量页面）* |
| `OTEL_RESOURCE_ATTRIBUTES` | 逗号分隔的 `key=value` 对，作为资源属性添加到 Claude Code 发出的所有 OpenTelemetry 度量数据点上。用于附加环境或部署标签（如 `environment=production,team=platform`），以便在收集器中过滤每个度量（v2.1.162） |
| `OTEL_LOG_USER_PROMPTS` | 设置为 `1` 以在 OpenTelemetry LLM 请求跨度中包含 `user_system_prompt` 字段。出于隐私原因默认省略 — 用户提示可能包含敏感数据，因此仅在你控制 OTel 收集器并有适当策略时选择加入 *（在 v2.1.121 变更日志中，尚未在官方环境变量页面）* |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | 用于度量和日志的 OpenTelemetry 收集器端点 URL。见 [监控](https://code.claude.com/docs/en/monitoring-usage) |
| `OTEL_EXPORTER_OTLP_HEADERS` | OpenTelemetry 导出器请求头（`Name=Value` 格式，逗号分隔），用于向收集器认证 |
| `OTEL_LOG_TOOL_CONTENT` | 设置为 `1` 以将完整的工具输入和输出作为 OpenTelemetry 日志事件发出。出于隐私原因默认省略 |
| `OTEL_METRICS_EXPORTER` | OpenTelemetry 度量导出器类型（如 `otlp`）。见 [监控](https://code.claude.com/docs/en/monitoring-usage) |
| `OTEL_TRACES_EXPORTER` | OpenTelemetry 追踪导出器类型（如 `otlp`）。见 [监控](https://code.claude.com/docs/en/monitoring-usage) |
| `OTEL_METRICS_INCLUDE_ENTRYPOINT` | 设置为 `1` 以将会话入口点（如交互式 vs `-p` vs SDK）作为标签包含在所有 OpenTelemetry 度量数据点上。适用于按 Claude Code 调用方式分解度量（v2.1.161 变更日志）*（在 v2.1.161 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_FORK_SUBAGENT` | 设置为 `1` 以在外部构建（非 Anthropic 签名分发版）上启用 fork 子 agent。Fork 子 agent 在隔离的子进程中运行，而非共享主 agent 的上下文 *（在 v2.1.117 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | MCP 服务器名称，作为环境变量传递给 `headersHelper` 脚本，以便它们可以生成特定于服务器的认证请求头 *（在 v2.1.85 变更日志中，尚未在官方环境变量页面）* |
| `CLAUDE_CODE_MCP_SERVER_URL` | MCP 服务器 URL，与 `CLAUDE_CODE_MCP_SERVER_NAME` 一起作为环境变量传递给 `headersHelper` 脚本 *（在 v2.1.85 变更日志中，尚未在官方环境变量页面）* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 覆盖 Opus 模型别名（如 `claude-opus-4-6[1m]`） |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上使用固定模型时，自定义 `/model` 选择器中 Opus 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Opus 条目的描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | 覆盖固定 Opus 模型的能力检测。以逗号分隔的值（如 `effort,thinking`）。当固定模型支持自动检测无法确认的功能时需要 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 覆盖 Sonnet 模型别名（如 `claude-sonnet-4-6`） |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上使用固定模型时，自定义 `/model` 选择器中 Sonnet 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Sonnet 条目的描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | 覆盖固定 Sonnet 模型的能力检测。以逗号分隔的值（如 `effort,thinking`）。当固定模型支持自动检测无法确认的功能时需要 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL` | 覆盖 Fable 模型别名（如 `claude-fable-5`） |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_NAME` | 自定义在 Bedrock/Vertex/Foundry 上使用固定模型时 `/model` 选择器中的 Fable 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中的 Fable 条目描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_SUPPORTED_CAPABILITIES` | 覆盖固定 Fable 模型的能力检测。以逗号分隔的值（如 `effort,thinking`）。当固定模型支持自动检测无法确认的功能时需要 |
| `MAX_THINKING_TOKENS` | 每次响应的最大扩展思考 token。设置为 `0` 以在 Anthropic API 上完全禁用扩展思考（等效于 `--thinking disabled`）。仅在使用固定思考预算时适用 — 在自适应思考模型（Opus 4.7+）上，努力级别控制思考深度 |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 设置用于自动压缩计算的 token 上下文容量。默认为模型的上下文窗口（标准 200K，扩展上下文模型 1M）。在 1M 模型上使用较低值（如 `500000`）将其视为 500K 用于压缩。上限为实际上下文窗口。`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 作为此值的百分比应用。设置此项将压缩阈值与状态行的 `used_percentage` 解耦 |
| `DISABLE_AUTO_COMPACT` | 禁用自动上下文压缩（`1` 禁用）。手动 `/compact` 仍然有效 *（不在官方文档中 — 未经验证）* |
| `DISABLE_COMPACT` | 禁用所有压缩 — 自动和手动（`1` 禁用） |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | 启用提示建议 |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | 要求会话使用计划模式 |
| `CLAUDE_CODE_TEAM_NAME` | agent 团队的团队名称 |
| `CLAUDE_CODE_TASK_LIST_ID` | 任务集成的任务列表 ID |
| `CLAUDE_ENV_FILE` | 自定义环境文件路径 |
| `FORCE_AUTOUPDATE_PLUGINS` | 强制插件自动更新（`1` 启用） |
| `HTTP_PROXY` | 网络请求的 HTTP 代理 URL |
| `HTTPS_PROXY` | 网络请求的 HTTPS 代理 URL |
| `NO_PROXY` | 绕过代理的主机逗号分隔列表 |
| `MCP_TOOL_TIMEOUT` | MCP 工具执行超时（毫秒） |
| `MCP_CLIENT_SECRET` | MCP OAuth 客户端密钥 |
| `MCP_OAUTH_CALLBACK_PORT` | MCP OAuth 回调端口 |
| `IS_DEMO` | 启用演示模式 |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | 斜杠命令工具输出的字符预算 |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Claude 3.5 Haiku 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Claude 3.7 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Claude 4.0 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Claude 4.0 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Claude 4.1 Opus 的 Vertex AI 区域覆盖 |

---

## 实用命令

| 命令 | 描述 |
|---------|-------------|
| `/model` | 切换模型并调整 Opus 4.6 努力级别 |
| `/effort` | 直接设置努力级别：`low`、`medium`、`high`、`xhigh`（仅 Opus 4.7，v2.1.111）或 `max`（仅 Opus 4.6）（v2.1.76+） |
| `/config` | 交互式配置 UI |
| `/memory` | 查看/编辑所有 memory 文件 |
| `/agents` | 管理子 agent |
| `/mcp` | 管理 MCP 服务器 |
| `/hooks` | 查看已配置的钩子 |
| `/plugin` | 管理插件 |
| `claude plugin tag` | 在市场中标记插件版本以进行分发。从市场仓库运行，带上插件名称和版本（v2.1.118） |
| `claude plugin prune` | 移除市场源不再存在的插件（如市场被删除或 `extraKnownMarketplaces` 条目被移除）。清理本地缓存并禁用孤立插件（v2.1.121） |
| `claude plugin details <plugin>` | 显示插件的组件清单（命令、agent、技能、钩子）以及它添加的每会话上下文 token 成本。适用于在托管环境中启用插件前审计 token 开销（v2.1.139） |
| `/keybindings` | 配置自定义键盘快捷键 |
| `/skills` | 查看和管理技能 |
| `/permissions` | 查看和管理权限规则 |
| `/usage-credits` | 查看剩余使用额度和限制。在 v2.1.144 中从 `/extra-usage` 重命名（旧名称仍然有效） |
| `--doctor` | 诊断配置问题 |
| `--debug` | 调试模式，包含钩子执行详情 |

---

## 快速参考：完整示例

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "advisorModel": "fable",
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
  "effortLevel": "high",
  "maxSkillDescriptionChars": 1536,
  "skillListingBudgetFraction": 0.01,
  "disableAgentView": false,
  "disableWorkflows": false,
  "workflowKeywordTriggerEnabled": true,
  "syntaxHighlightingDisabled": false,

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"],
    "baseRef": "fresh",
    "bgIsolation": "worktree"
  },

  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ],
    "soft_deny": ["$defaults", "Never run terraform apply"],
    "hard_deny": ["Never run rm -rf on directories outside the project"]
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
    "ANTHROPIC_BEDROCK_SERVICE_TIER": "priority",
    "CLAUDE_CODE_ENABLE_AUTO_MODE": "1"
  }
}
```

---

## 来源

- [Claude Code 设置文档](https://code.claude.com/docs/en/settings)
- [Claude Code 设置 JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code 变更日志](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub 设置示例](https://github.com/feiskyer/claude-code-settings)
- [Claude Code 环境变量参考](https://code.claude.com/docs/en/env-vars)
- [Claude Code 权限参考](https://code.claude.com/docs/en/permissions)

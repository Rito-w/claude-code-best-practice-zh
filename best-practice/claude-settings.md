# Settings 最佳实践


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

**重要说明**：
- `deny` 规则具有最高安全优先级，不能被低优先级的 allow/ask 规则覆盖。
- 托管设置可能锁定或覆盖本地行为，即使本地文件指定了不同的值。
- 数组设置（如 `permissions.allow`）在各范围之间**合并并去重** — 所有层级的条目被组合在一起，而非替换。

---

## 核心配置

### 通用设置

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|

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

### 工具权限语法

| 工具 | 语法 | 示例 |
|------|--------|----------|

**评估顺序：** 规则按以下顺序评估：首先是 deny 规则，然后是 ask，最后是 allow。第一个匹配的规则生效。


**读/编辑路径模式：** `Read`、`Edit` 和 `Write` 的权限规则支持 gitignore 风格的模式，带有四种前缀类型：

| 前缀 | 含义 | 示例 |
|--------|---------|---------|
| `//` | 从文件系统根目录开始的绝对路径 | `Read(//Users/alice/file)` |
| `~/` | 相对于主目录 | `Read(~/.zshrc)` |
| `/` | 相对于项目根目录 | `Edit(/src/**)` |
| `./` 或无前缀 | 相对路径（当前目录） | `Read(.env)`、`Read(*.ts)` |


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


### MCP 设置

| 键 | 类型 | 范围 | 描述 |
|-----|------|-------|-------------|

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

**示例：**
```json
{
  "model": "opus"
}
```


### 模型覆盖

将 Anthropic 模型 ID 映射到 Bedrock、Vertex 或 Foundry 部署的特定提供商模型 ID。

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|

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

**使用方法：**
1. 运行 `/effort low`、`/effort medium` 或 `/effort high` 直接设置（v2.1.76+）
2. 或运行 `/model` → 选择模型 → 使用 **← →** 方向键调整
3. 设置通过 `settings.json` 中的 `effortLevel` 键持久化


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

| `preferredNotifChannel` | string | `"auto"` | 任务完成和权限提示通知的方式。值：`"auto"`、`"terminal_bell"`、`"iterm2"`、`"iterm2_with_bell"`、`"kitty"`、`"ghostty"`、`"notifications_disabled"`。默认 `"auto"` 在 iTerm2、Ghostty 和 Kitty 中发送桌面通知，在其他终端中不执行任何操作。设置 `"terminal_bell"` 可在任何终端中响铃字符。在 `/config` 中显示为 **Notifications**。见 [获取终端铃或通知](https://code.claude.com/docs/en/terminal-config#get-a-terminal-bell-or-notification) |

### 全局配置设置（`~/.claude.json`）

这些 IDE 相关偏好存储在 `~/.claude.json` 中，**而非** `settings.json`。

> **v2.1.119 迁移说明：** 截至 v2.1.119，`autoScrollEnabled`、`editorMode`、`showTurnDuration`、`teammateMode` 和 `terminalProgressBarEnabled` 已移入 `settings.json` 并在上方的显示设置表中有文档说明。早期版本将它们存储在此处。

| 键 | 类型 | 默认 | 描述 |
|-----|------|---------|-------------|

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

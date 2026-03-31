<!-- 翻译标记：.claude/hooks/HOOKS-README.md - 已翻译 -->
# HOOKS-README

包含 Hooks 的所有详细信息、脚本和指令。

## Hook Events 概述 - [官方 22 个 Hooks](https://code.claude.com/docs/en/hooks)

Claude Code 提供多个在工作流不同时间点运行的 Hook Events：

| # | Hook | 描述 | 选项 |
|:-:|------|------|------|
| 1 | `PreToolUse` | 在工具调用前运行（可以阻止它们） | `async`, `timeout: 5000`, `tool_use_id` |
| 2 | `PermissionRequest` | 当 Claude Code 向用户请求权限时运行 | `async`, `timeout: 5000`, `permission_suggestions` |
| 3 | `PostToolUse` | 在工具调用成功完成后运行 | `async`, `timeout: 5000`, `tool_response`, `tool_use_id` |
| 4 | `PostToolUseFailure` | 在工具调用失败后运行 | `async`, `timeout: 5000`, `error`, `is_interrupt`, `tool_use_id` |
| 5 | `UserPromptSubmit` | 当用户提交提示时运行，在 Claude 处理之前 | `async`, `timeout: 5000`, `prompt` |
| 6 | `Notification` | 当 Claude Code 发送通知时运行 | `async`, `timeout: 5000`, `notification_type`, `message`, `title` |
| 7 | `Stop` | 当 Claude Code 完成响应时运行 | `async`, `timeout: 5000`, `last_assistant_message`, `stop_hook_active` |
| 8 | `SubagentStart` | 当 Subagent 任务开始时运行 | `async`, `timeout: 5000`, `agent_id`, `agent_type` |
| 9 | `SubagentStop` | 当 Subagent 任务完成时运行 | `async`, `timeout: 5000`, `agent_id`, `agent_type`, `last_assistant_message`, `agent_transcript_path`, `stop_hook_active` |
| 10 | `PreCompact` | 在 Claude Code 即将运行压缩操作前运行 | `async`, `timeout: 5000`, `once`, `trigger`, `custom_instructions` |
| 11 | `PostCompact` | 在 Claude Code 完成压缩操作后运行 | `async`, `timeout: 5000`, `trigger`, `compact_summary` |
| 12 | `SessionStart` | 当 Claude Code 启动新会话或恢复现有会话时运行 | `async`, `timeout: 5000`, `once`, `agent_type`, `model`, `source` |
| 13 | `SessionEnd` | 当 Claude Code 会话结束时运行 | `async`, `timeout: 5000`, `once`, `reason` |
| 14 | `Setup` | 当 Claude Code 运行 /setup 命令进行项目初始化时运行 | `async`, `timeout: 30000` |
| 15 | `TeammateIdle` | 当 Teammate Agent 变为空闲时运行（实验性 Agent Teams） | `async`, `timeout: 5000`, `teammate_name`, `team_name` |
| 16 | `TaskCompleted` | 当后台任务完成时运行（实验性 Agent Teams） | `async`, `timeout: 5000`, `task_id`, `task_subject`, `task_description`, `teammate_name`, `team_name` |
| 17 | `ConfigChange` | 当会话期间配置文件更改时运行 | `async`, `timeout: 5000`, `file_path`, `source` |
| 18 | `WorktreeCreate` | 当 Agent Worktree 隔离为自定义 VCS 设置创建 Worktrees 时运行 | `async`, `timeout: 5000`, `name` |
| 19 | `WorktreeRemove` | 当 Agent Worktree 隔离为自定义 VCS 拆除移除 Worktrees 时运行 | `async`, `timeout: 5000`, `worktree_path` |
| 20 | `InstructionsLoaded` | 当 CLAUDE.md 或 `.claude/rules/*.md` 文件加载到 Context 时运行 | `async`, `timeout: 5000`, `file_path`, `memory_type`, `load_reason`, `globs`, `trigger_file_path`, `parent_file_path` |
| 21 | `Elicitation` | 当 MCP Server 在工具调用期间请求用户输入时运行 | `async`, `timeout: 5000`, `mcp_server_name`, `message`, `mode`, `url`, `elicitation_id`, `requested_schema` |
| 22 | `ElicitationResult` | 当用户响应 MCP Elicitation 后运行，在响应发送回 Server 之前 | `async`, `timeout: 5000`, `mcp_server_name`, `action`, `content`, `mode`, `elicitation_id` |

> **注意：** Hooks 15-16（`TeammateIdle` 和 `TaskCompleted`）需要实验性 Agent Teams 功能。启动 Claude Code 时设置 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 启用它们。

### 不在官方文档中

以下项目存在于 [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) 中但**未列出**在 [官方 Hooks Reference](https://code.claude.com/docs/en/hooks) 中：

| 项目 | 添加于 | Changelog Reference | 注意 |
|------|--------|-------------------|------|
| `Setup` hook | [v2.1.10](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md#2110) | "添加了新的 Setup Hook Event，可通过 `--init`, `--init-only`, 或 `--maintenance` CLI Flags 触发，用于仓库设置和维护操作" | 官方 Hooks Reference 页面未列出（列出 21 个 Hooks，不包括 Setup） |
| Agent Frontmatter Hooks | [v2.1.0](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md#210) | "在 Agent Frontmatter 中添加 Hooks 支持，允许 Agents 定义范围限定到 Agent 生命周期的 PreToolUse, PostToolUse, 和 Stop Hooks" | Changelog 仅提到 3 个 Hooks，但测试确认 **6 个 Hooks** 实际上在 Agent 会话中触发：PreToolUse, PostToolUse, PermissionRequest, PostToolUseFailure, Stop, SubagentStop。并非所有 16 个 Hooks 都支持。 |

## 先决条件

使用 Hooks 之前，确保你的系统上安装了 **Python 3**。

### 所需软件

#### 所有平台（Windows, macOS, Linux）
- **Python 3**: 运行 Hook 脚本所需
- 验证安装：`python3 --version`

**安装说明：**
- **Windows**: 从 [python.org](https://www.python.org/downloads/) 下载或通过 `winget install Python.Python.3` 安装
- **macOS**: 通过 `brew install python3` 安装（需要 [Homebrew](https://brew.sh/)）
- **Linux**: 通过 `sudo apt install python3` 安装（Ubuntu/Debian）或 `sudo yum install python3`（RHEL/CentOS）

### 音频播放器（可选 - 自动检测）

Hook 脚本自动检测并使用适合你平台的音频播放器：

- **macOS**: 使用 `afplay`（内置，无需安装）
- **Linux**: 使用 `paplay` 来自 `pulseaudio-utils` - 通过 `sudo apt install pulseaudio-utils` 安装
- **Windows**: 使用内置 `winsound` 模块（随 Python 包含）

### Hooks 如何执行

Hooks 在 `.claude/settings.json` 中配置为直接使用 Python 3 运行：

```json
{
  "type": "command",
  "command": "python3 .claude/hooks/scripts/hooks.py"
}
```

## 配置 Hooks（启用/禁用）

Hooks 可以全局和单独级别轻松启用或禁用。

### 一次性禁用所有 Hooks

编辑 `.claude/settings.local.json` 并设置：
```json
{
  "disableAllHooks": true
}
```

**注意：** `.claude/settings.local.json` 文件是 git-ignored 的，因此每个开发者可以配置自己的 Hook 偏好而不影响团队的共享设置在 `.claude/settings.json` 中。

> **Managed Settings:** 如果管理员通过 Managed Policy 设置配置了 Hooks，在用户、项目或本地设置中设置的 `disableAllHooks` 无法禁用这些 Managed Hooks（v2.1.49 中修复）。

### 禁用单个 Hooks

对于细粒度控制，你可以通过编辑 Hooks 配置文件禁用特定 Hooks。

#### 配置文件

有两个配置文件用于管理单个 Hooks：

1. **`.claude/hooks/config/hooks-config.json`** - 提交到 git 的共享/默认配置
2. **`.claude/hooks/config/hooks-config.local.json`** - 你的个人覆盖（git-ignored）

本地配置文件（`.local.json`）优先于共享配置，允许每个开发者自定义 Hook 行为而不影响团队。

#### 共享配置

编辑 `.claude/hooks/config/hooks-config.json` 用于团队范围默认值：

```json
{
  "disableLogging": false,
  "disablePreToolUseHook": false,
  "disablePermissionRequestHook": false,
  "disablePostToolUseHook": false,
  "disablePostToolUseFailureHook": false,
  "disableUserPromptSubmitHook": false,
  "disableNotificationHook": false,
  "disableStopHook": false,
  "disableSubagentStartHook": false,
  "disableSubagentStopHook": false,
  "disablePreCompactHook": false,
  "disablePostCompactHook": false,
  "disableElicitationHook": false,
  "disableElicitationResultHook": false,
  "disableSessionStartHook": false,
  "disableSessionEndHook": false,
  "disableSetupHook": false,
  "disableTeammateIdleHook": false,
  "disableTaskCompletedHook": false,
  "disableConfigChangeHook": false,
  "disableWorktreeCreateHook": false,
  "disableWorktreeRemoveHook": false,
  "disableInstructionsLoadedHook": false
}
```

**配置选项：**
- `disableLogging`: 设为 `true` 禁用将 Hook Events 记录到 `.claude/hooks/logs/hooks-log.jsonl`（用于防止日志文件增长）

#### 本地配置（个人覆盖）

创建或编辑 `.claude/hooks/config/hooks-config.local.json` 用于个人偏好：

```json
{
  "disableLogging": true,
  "disablePostToolUseHook": true,
  "disableSessionStartHook": true
}
```

在此示例中，日志记录被禁用，PostToolUse 和 SessionStart Hooks 在本地覆盖。所有其他 Hooks 将使用共享配置值。

**注意：** 单个 Hook 切换由 Hook 脚本（`.claude/hooks/scripts/hooks.py`）检查。本地设置覆盖共享设置，如果 Hook 被禁用，脚本静默退出而不播放任何声音或执行 Hook 逻辑。

### Text to Speech (TTS)

用于生成声音的网站：https://elevenlabs.io/

使用的声音：Samara X

## Agent Frontmatter Hooks

Claude Code 2.1.0 引入了对 Agent Frontmatter 文件中定义的 Agent 特定 Hooks 的支持。这些 Hooks 仅在 Agent 生命周期内运行并支持 Hook Events 的子集。

### 支持的 Agent Hooks

Agent Frontmatter Hooks 支持 **6 个 Hooks**（不是全部 16 个）。Changelog 最初只提到 3 个，但测试确认 6 个 Hooks 实际上在 Agent 会话中触发：
- `PreToolUse`: 在 Agent 使用工具前运行
- `PostToolUse`: 在 Agent 完成工具使用后运行
- `PermissionRequest`: 当工具需要用户权限时运行
- `PostToolUseFailure`: 在工具调用失败后运行
- `Stop`: 当 Agent 完成时运行
- `SubagentStop`: 当 Subagent 完成时运行

> **注意：** [v2.1.0 changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md#210) 只提到 3 个 Hooks: *"Added hooks support to agent frontmatter, allowing agents to define PreToolUse, PostToolUse, and Stop hooks scoped to the agent's lifecycle"*。然而，使用 `claude-code-voice-hook-agent` 测试确认 6 个 Hooks 实际上在 Agent 会话中触发。剩余 10 个 Hooks（例如 Notification, SessionStart, SessionEnd 等）不在 Agent Context 中触发。
>
> **更新（2026 年 2 月）：** [官方 Hooks Reference](https://code.claude.com/docs/en/hooks) 现在说明 *"All hook events are supported"* 用于 Skill/Agent Frontmatter Hooks。这可能意味着支持已扩展到最初测试的 6 个 Hooks 之外。建议重新测试以验证是否有额外 Hooks 现在在 Agent 会话中触发。

### Agent 声音文件夹

Agent 特定声音存储在单独的文件夹中：
- `.claude/hooks/sounds/agent_pretooluse/`
- `.claude/hooks/sounds/agent_posttooluse/`
- `.claude/hooks/sounds/agent_permissionrequest/`
- `.claude/hooks/sounds/agent_posttoolusefailure/`
- `.claude/hooks/sounds/agent_stop/`
- `.claude/hooks/sounds/agent_subagentstop/`

### 创建带有 Hooks 的 Agent

1. 在 `.claude/agents/` 中创建 Agent 定义文件：

```markdown
---
name: my-agent
description: 此 Agent 的功能描述
hooks:
  PreToolUse:
    - type: command
      command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=my-agent
      timeout: 5000
      async: true
      statusMessage: PreToolUse
  PostToolUse:
    - type: command
      command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=my-agent
      timeout: 5000
      async: true
      statusMessage: PostToolUse
  PermissionRequest:
    - type: command
      command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=my-agent
      timeout: 5000
      async: true
      statusMessage: PermissionRequest
  PostToolUseFailure:
    - type: command
      command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=my-agent
      timeout: 5000
      async: true
      statusMessage: PostToolUseFailure
  Stop:
    - type: command
      command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=my-agent
      timeout: 5000
      async: true
      statusMessage: Stop
  SubagentStop:
    - type: command
      command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=my-agent
      timeout: 5000
      async: true
      statusMessage: SubagentStop
---

你的 Agent 指令在此...
```

2. 将声音文件添加到 Agent 声音文件夹：
   - `agent_pretooluse/agent_pretooluse.wav`
   - `agent_posttooluse/agent_posttooluse.wav`
   - `agent_permissionrequest/agent_permissionrequest.wav`
   - `agent_posttoolusefailure/agent_posttoolusefailure.wav`
   - `agent_stop/agent_stop.wav`
   - `agent_subagentstop/agent_subagentstop.wav`

### 示例：Weather Fetcher Agent

完整示例见 `.claude/agents/claude-code-voice-hook-agent.md` 带有配置 Hooks 的 Agent。

### Hook 选项：`once: true`

`once: true` 选项确保 Hook 每个会话只运行一次：

```json
{
  "type": "command",
  "command": "python3 .claude/hooks/scripts/hooks.py",
  "timeout": 5000,
  "once": true
}
```

这对于像 `SessionStart`, `SessionEnd`, 和 `PreCompact` 这样应该只触发一次的 Hooks 很有用。

> **注意：** `once` 选项**仅适用于 Skills，不适用于 Agents**。它在基于设置的 Hooks 和 Skill Frontmatter 中有效，但不支持 Agent Frontmatter Hooks。

### Hook 选项：`async: true`

Hooks 可以通过添加 `"async": true` 在后台运行而不阻塞 Claude Code 的执行：

```json
{
  "type": "command",
  "command": "python3 .claude/hooks/scripts/hooks.py",
  "timeout": 5000,
  "async": true
}
```

**何时使用异步 Hooks：**
- 日志记录和分析
- 通知和声音效果
- 任何不应减慢 Claude Code 的副作用

此项目对所有 Hooks 使用 `async: true`，因为语音通知是不需要阻塞执行的副作用。`timeout` 指定异步 Hook 在终止前可以运行多长时间。

### Hook 选项：`statusMessage`

`statusMessage` 字段设置 Hook 运行时向用户显示的自定义 Spinner 消息：

```json
{
  "type": "command",
  "command": "python3 .claude/hooks/scripts/hooks.py",
  "timeout": 5000,
  "async": true,
  "statusMessage": "PreToolUse"
}
```

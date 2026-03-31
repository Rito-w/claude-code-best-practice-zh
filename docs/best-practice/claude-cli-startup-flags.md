<!-- 翻译标记：best-practice/claude-cli-startup-flags.md - 已翻译 -->
# CLI Startup Flags 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

从 Terminal 启动 Claude Code 时的启动 Flags、顶级 Subcommands 和启动 Environment Variables 参考。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 目录

1. [Session Management](#session-management)
2. [Model & Configuration](#model--configuration)
3. [Permissions & Security](#permissions--security)
4. [Output & Format](#output--format)
5. [System Prompt](#system-prompt)
6. [Agent & Subagent](#agent--subagent)
7. [MCP & Plugins](#mcp--plugins)
8. [Directory & Workspace](#directory--workspace)
9. [Budget & Limits](#budget--limits)
10. [Integration](#integration)
11. [Initialization & Maintenance](#initialization--maintenance)
12. [Debug & Diagnostics](#debug--diagnostics)
13. [Settings Override](#settings-override)
14. [Version & Help](#version--help)
15. [Subcommands](#subcommands)
16. [Environment Variables](#environment-variables)

---

## Session Management

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--continue` | `-c` | 继续当前目录中最近的会话 |
| `--resume` | `-r` | 通过 ID 或名称恢复特定会话，或显示交互式选择器 |
| `--from-pr <NUMBER\|URL>` | | 恢复链接到特定 GitHub PR 的会话 |
| `--fork-session` | | 恢复时创建新的会话 ID（与 `--resume` 或 `--continue` 一起使用） |
| `--session-id <UUID>` | | 使用特定会话 ID（必须是有效 UUID） |
| `--no-session-persistence` | | 禁用会话持久化（仅 Print Mode） |
| `--remote` | | 在 claude.ai 上创建新的 Web 会话 |
| `--teleport` | | 在本地 Terminal 中恢复 Web 会话 |

---

## Model & Configuration

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--model <NAME>` | | 设置 Model 使用别名（`sonnet`, `opus`, `haiku`）或完整 Model ID |
| `--fallback-model <NAME>` | | 当默认 Model 过载时自动回退（仅 Print Mode） |
| `--betas <LIST>` | | 在 API 请求中包含的 Beta Headers（仅限 API Key 用户） |

---

## Permissions & Security

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--dangerously-skip-permissions` | | 跳过所有权限提示。极度谨慎使用 |
| `--allow-dangerously-skip-permissions` | | 启用权限绕过作为选项但不激活 |
| `--permission-mode <MODE>` | | 以指定权限模式开始：`default`, `plan`, `acceptEdits`, `bypassPermissions` |
| `--allowedTools <TOOLS>` | | 无需提示即可执行的工具（权限规则语法） |
| `--disallowedTools <TOOLS>` | | 从 Model Context 中完全移除的工具 |
| `--tools <TOOLS>` | | 限制 Claude 可以使用的内置工具（使用 `""` 禁用所有） |
| `--permission-prompt-tool <TOOL>` | | 指定 MCP 工具在非交互模式下处理权限提示 |

---

## Output & Format

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--print` | `-p` | 打印响应而不进入交互模式（Headless/SDK Mode） |
| `--output-format <FORMAT>` | | 输出格式：`text`, `json`, `stream-json` |
| `--input-format <FORMAT>` | | 输入格式：`text`, `stream-json` |
| `--json-schema <SCHEMA>` | | 获取匹配 Schema 的已验证 JSON（仅 Print Mode） |
| `--include-partial-messages` | | 包含部分流式事件（需要 `--print` 和 `--output-format=stream-json`） |
| `--verbose` | | 启用详细日志记录，包含完整的每轮输出 |

---

## System Prompt

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--system-prompt <TEXT>` | | 用自定义文本替换整个 System Prompt |
| `--system-prompt-file <PATH>` | | 从文件加载 System Prompt，替换默认（仅 Print Mode） |
| `--append-system-prompt <TEXT>` | | 将自定义文本附加到默认 System Prompt |
| `--append-system-prompt-file <PATH>` | | 将文件内容附加到默认 Prompt（仅 Print Mode） |

---

## Agent & Subagent

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--agent <NAME>` | | 为当前会话指定 Agent |
| `--agents <JSON>` | | 通过 JSON 动态定义自定义 Subagents |
| `--teammate-mode <MODE>` | | 设置 Agent Team 显示：`auto`, `in-process`, `tmux` |

---

## MCP & Plugins

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--mcp-config <PATH\|JSON>` | | 从 JSON 文件或字符串加载 MCP Servers |
| `--strict-mcp-config` | | 仅使用 `--mcp-config` 中的 MCP Servers，忽略所有其他 |
| `--plugin-dir <PATH>` | | 仅为此会话从目录加载插件（可重复） |

---

## Directory & Workspace

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--add-dir <PATH>` | | 添加 Claude 可访问的额外工作目录 |
| `--worktree` | `-w` | 在隔离的 Git Worktree 中启动 Claude（从 HEAD 分支） |

---

## Budget & Limits

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--max-budget-usd <AMOUNT>` | | API 调用的最大美元金额，达到后停止（仅 Print Mode） |
| `--max-turns <NUMBER>` | | 限制 Agentic 轮数（仅 Print Mode） |

---

## Integration

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--chrome` | | 启用 Chrome 浏览器集成用于 Web 自动化 |
| `--no-chrome` | | 为此会话禁用 Chrome 浏览器集成 |
| `--ide` | | 如果恰好有一个有效 IDE 可用，启动时自动连接 |

---

## Initialization & Maintenance

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--init` | | 运行初始化 Hooks 并启动交互模式 |
| `--init-only` | | 运行初始化 Hooks 并退出（无交互会话） |
| `--maintenance` | | 运行维护 Hooks 并退出 |

---

## Debug & Diagnostics

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--debug <CATEGORIES>` | | 启用调试模式并可选类别过滤（例如 `"api,hooks"`） |

---

## Settings Override

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--settings <PATH\|JSON>` | | 要加载的设置 JSON 文件路径或 JSON 字符串 |
| `--setting-sources <LIST>` | | 要加载的源逗号分隔列表：`user`, `project`, `local` |
| `--disable-slash-commands` | | 为此会话禁用所有 Skills 和 Slash Commands |

---

## Version & Help

| Flag | 简写 | 描述 |
|------|-------|-------------|
| `--version` | `-v` | 输出版本号 |
| `--help` | `-h` | 显示帮助信息 |

---

## Subcommands

这些是作为 `claude <subcommand>` 运行的顶级命令：

| Subcommand | 描述 |
|------------|------|
| `claude` | 启动交互式 REPL |
| `claude "query"` | 使用初始提示启动 REPL |
| `claude agents` | 列出配置的 Agents |
| `claude auth` | 管理 Claude Code 认证 |
| `claude doctor` | 从命令行运行诊断 |
| `claude install` | 安装或切换 Claude Code 原生构建 |
| `claude mcp` | 配置 MCP Servers（`add`, `remove`, `list`, `get`, `enable`） |
| `claude plugin` | 管理 Claude Code 插件 |
| `claude remote-control` | 管理远程控制会话 |
| `claude setup-token` | 创建用于订阅使用的长生命周期 Token |
| `claude update` / `claude upgrade` | 更新到最新版本 |

---

## Environment Variables

这些仅限启动的 Environment Variables 在启动 Claude Code 之前在 Shell 中设置（它们不能通过 `settings.json` 配置）：

| Variable | 描述 |
|----------|------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 启用实验性 Agent Teams |
| `CLAUDE_CODE_TMPDIR` | 覆盖内部文件的临时目录 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | 启用额外目录 CLAUDE.md 加载 |
| `DISABLE_AUTOUPDATER=1` | 禁用自动更新 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 控制思考深度 — 见 [Settings Reference](./claude-settings.md#environment-variables-via-env) |
| `USE_BUILTIN_RIPGREP=0` | 使用系统 ripgrep 而非内置（Alpine Linux） |
| `CLAUDE_CODE_SIMPLE` | 启用简单模式（仅 Bash + Edit 工具） |
| `CLAUDE_BASH_NO_LOGIN=1` | 跳过 BashTool 的登录 Shell |

对于可通过 `settings.json` 中的 `"env"` 键配置的 Environment Variables（包括 `MAX_THINKING_TOKENS`, `CLAUDE_CODE_SHELL`, `CLAUDE_CODE_ENABLE_TASKS`, `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`, `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` 等），见 [Claude Settings Reference](./claude-settings.md#environment-variables-via-env)。

---

## 来源

- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Headless Mode](https://code.claude.com/docs/en/headless)
- [Claude Code Setup](https://code.claude.com/docs/en/setup)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code Common Workflows](https://code.claude.com/docs/en/common-workflows)

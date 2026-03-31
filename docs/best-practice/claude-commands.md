<!-- TRANSLATED: Auto-translated from source -->

# Commands Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2028%2C%202026%206%3A05%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-commands-implementation.md)

Claude Code commands — frontmatter fields 和 official built-in slash commands。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (13)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Display name 和 `/slash-command` identifier。如果省略则默认为 directory name |
| `description` | string | Recommended | Command 的作用。在 autocomplete 中显示并被 Claude 用于 auto-discovery |
| `argument-hint` | string | No | 在 autocomplete 期间显示的 hint（例如 `[issue-number]`, `[filename]`） |
| `disable-model-invocation` | boolean | No | 设置为 `true` 以防止 Claude 自动调用此 command |
| `user-invocable` | boolean | No | 设置为 `false` 以在 `/` menu 中隐藏 — command 变为仅 background knowledge |
| `paths` | string/list | No | Glob patterns 限制此 skill 何时激活。接受逗号分隔的 string 或 YAML list。设置后，Claude 仅在处理匹配文件时自动加载 skill |
| `allowed-tools` | string | No | 当此 command 激活时允许使用的 Tools（无需 permission prompts） |
| `model` | string | No | 当此 command 运行时使用的 Model（例如 `haiku`, `sonnet`, `opus`） |
| `effort` | string | No | 调用时覆盖 model effort level（`low`, `medium`, `high`, `max`） |
| `context` | string | No | 设置为 `fork` 以在 isolated subagent context 中运行 command |
| `agent` | string | No | 当设置 `context: fork` 时的 Subagent type（默认：`general-purpose`） |
| `shell` | string | No | `` !`command` `` blocks 的 Shell — 接受 `bash`（默认）或 `powershell`。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | No | 作用域为此 command 的 Lifecycle hooks |

---

## ![Official](_media/tags/official.svg) **(64)**

| # | Command | Tag | Description |
|---|---------|-----|-------------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过 OAuth 认证 Claude Code |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 退出 Claude Code |
| 3 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 打开升级页面切换到更高级别的计划 |
| 4 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置当前会话的 prompt bar 颜色 |
| 5 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开设置界面调整主题、model、output style 和其他偏好。Alias: `/settings` |
| 6 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 自定义每个 context 的 keyboard shortcuts 并创建 chord sequences |
| 7 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 查看或更新 tool permissions。Alias: `/allowed-tools` |
| 8 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 管理 privacy 和 telemetry preferences |
| 9 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置 sandboxing 并查看 dependency status |
| 10 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置 Claude Code 的 status line UI |
| 11 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 订购 Claude Code stickers |
| 12 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 在 IDE terminals 中启用 shift+enter 换行 |
| 13 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 更改颜色主题 |
| 14 | `/vim` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 启用 vim-style editing mode |
| 15 | `/voice` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换 push-to-talk voice dictation。需要 Claude.ai account |
| 16 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 以彩色网格和 token counts 可视化当前 context usage |
| 17 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 显示当前会话的 token usage statistics |
| 18 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 配置 subscription plans 的 pay-as-you-go overflow billing |
| 19 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 生成分析报告，分析你的 Claude Code sessions，包括 project areas、interaction patterns 和 friction points |
| 20 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 可视化 daily usage、session history、streaks 和 model preferences |
| 21 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 打开设置界面（Status tab）显示 version、model、account 和 connectivity |
| 22 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 显示 plan usage limits 和 rate limit status（仅限 subscription plans） |
| 23 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 检查 Claude Code installation 的健康状态 |
| 24 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 提交关于 Claude Code 的反馈。Alias: `/bug` |
| 25 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 显示 slash-command help |
| 26 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 显示最近的 Claude Code release notes |
| 27 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 列出和管理 background tasks |
| 28 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 复制最近（或第 N 个最近）的 assistant response 到剪贴板。为 code blocks 显示交互式选择器 |
| 29 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 导出当前对话到文件或剪贴板 |
| 30 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 custom subagents — view, create, edit, delete |
| 31 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 Claude in Chrome browser integration |
| 32 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 tool events 的 hook configurations |
| 33 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 连接到 IDE integration |
| 34 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 MCP server connections |
| 35 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 Claude Code plugins |
| 36 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 重新加载已安装的 plugins 而无需重启 |
| 37 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 列出可用的 skills |
| 38 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | 编辑 CLAUDE.md memory files，启用或禁用 auto-memory，查看 auto-memory entries |
| 39 | `/effort [low\|medium\|high\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 设置 model effort level |
| 40 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 切换 fast mode — 相同的 Opus 4.6 model 但输出更快 |
| 41 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 选择或更改 AI model |
| 42 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 与朋友分享一周免费 Claude Code。仅当你的 account 符合条件时可见 |
| 43 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 直接从 prompt 进入 plan mode。传递可选 description 立即开始该任务 |
| 44 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 为当前会话添加新的 working directory |
| 45 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 打开交互式 diff viewer 显示 uncommitted changes 和 per-turn diffs |
| 46 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 用 CLAUDE.md guide 初始化新项目 |
| 47 | `/pr-comments [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 获取并显示 GitHub pull request 的评论。自动检测当前分支的 PR，或传递 PR URL 或编号 |
| 48 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 已弃用 — 改用 `code-review` plugin |
| 49 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 对当前更改运行 focused security review |
| 50 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在 Claude Code Desktop app 中继续当前会话。仅限 macOS 和 Windows。Alias: `/app` |
| 51 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 安装 GitHub app 用于 PR-linked workflows |
| 52 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 安装 Slack app 用于 notifications 和 sharing |
| 53 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 显示 QR code 下载 Claude mobile app。Aliases: `/ios`, `/android` |
| 54 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使此会话可从 claude.ai 进行 remote control。Alias: `/rc` |
| 55 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 检查或复制 remote-control environment setup |
| 56 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 创建、更新、列出或运行 Cloud scheduled tasks。Claude 通过对话引导你完成设置 |
| 57 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 在当前点创建当前对话的分支。Alias: `/fork` |
| 58 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 询问快速侧边问题而不添加到对话中 |
| 59 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 清除对话历史并释放 context。Aliases: `/reset`, `/new` |
| 60 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Compact conversation 并可选 focus instructions |
| 61 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 退出 CLI。Alias: `/quit` |
| 62 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 重命名当前会话。不指定名称时从对话历史自动生成 |
| 63 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 通过 ID 或名称恢复之前的对话，或打开 session picker。Alias: `/continue` |
| 64 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将对话和/或代码回滚到之前的点，或从选定消息开始总结。Alias: `/checkpoint` |

Bundled skills 如 `/debug` 也可能出现在 slash-command menu 中，但它们不是 built-in commands。

---

## Sources

- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

# Commands 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2026%2C%202026%201%3A10%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.119-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-commands-implementation.md)

Claude Code commands —— frontmatter 字段和官方内置 slash commands。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter 字段 (15)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | 否 | 显示名称和 `/slash-command` 标识符。如果省略，默认为目录名 |
| `description` | string | 推荐 | Command 的作用。显示在自动补全中，供 Claude 用于自动发现 |
| `when_to_use` | string | 否 | 关于 Claude 何时应调用该 Command 的额外上下文 —— 触发短语和请求示例。附加到 `description` 之后显示在列表中，计入 1,536 字符上限 |
| `argument-hint` | string | 否 | 自动补全期间显示的提示（如 `[issue-number]`、`[filename]`） |
| `arguments` | string/list | 否 | 用于在 Command 内容中进行 `$name` 替换的命名位置参数。接受以空格分隔的字符串或 YAML 列表 —— 名称按顺序映射到参数位置 |
| `disable-model-invocation` | boolean | 否 | 设为 `true` 可阻止 Claude 自动调用此 Command |
| `user-invocable` | boolean | 否 | 设为 `false` 可从 `/` 菜单中隐藏 —— Command 仅作为背景知识使用 |
| `paths` | string/list | 否 | 限制此 Command 何时激活的 glob 模式。接受逗号分隔的字符串或 YAML 列表。设置后，Claude 仅在处理匹配文件时加载该 Command |
| `allowed-tools` | string | 否 | 此 Command 激活时无需权限提示即可使用的工具 |
| `model` | string | 否 | 运行此 Command 时使用的模型（如 `haiku`、`sonnet`、`opus`） |
| `effort` | string | 否 | 调用时覆盖模型的努力级别（`low`、`medium`、`high`、`max`） |
| `context` | string | 否 | 设为 `fork` 可在隔离的 subagent 上下文中运行 Command |
| `agent` | string | 否 | 当 `context: fork` 时使用的 subagent 类型（默认：`general-purpose`） |
| `shell` | string | 否 | `` !`command` `` 块使用的 shell —— 接受 `bash`（默认）或 `powershell`。需要设置 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | 否 | 作用于此 Command 的生命周期 Hook |

---

## ![Official](../!/tags/official.svg) **(75)**

| # | Command | Tag | Description |
|---|---------|-----|-------------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 登录你的 Anthropic 账户 |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 退出你的 Anthropic 账户 |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Amazon Bedrock 认证、区域和模型固定。仅在设置 `CLAUDE_CODE_USE_BEDROCK=1` 时可见。首次使用 Bedrock 的用户也可以从登录界面进入此向导 |
| 4 | `/setup-vertex` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Google Vertex AI 认证、项目、区域和模型固定。仅在设置 `CLAUDE_CODE_USE_VERTEX=1` 时可见。首次使用 Vertex AI 的用户也可以从登录界面进入此向导 |
| 5 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 打开升级页面以切换到更高级的套餐 |
| 6 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置当前会话的 prompt 栏颜色。可用颜色：`red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink`、`cyan`。使用 `default` 重置 |
| 7 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开设置界面以调整主题、模型、输出样式和其他偏好。别名：`/settings` |
| 8 | `/focus` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换专注视图，仅显示最后一个 prompt、工具调用摘要和最终回复。有助于减少长会话中的视觉干扰。仅在全屏渲染中可用 |
| 9 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开或创建你的快捷键配置文件 |
| 10 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 管理工具权限的允许、询问和拒绝规则。打开交互式对话框，可按范围查看规则、添加或删除规则、管理工作目录以及查看最近的 auto mode 拒绝记录。别名：`/allowed-tools` |
| 11 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 查看和更新你的隐私设置。仅对 Pro 和 Max 套餐订阅者可用 |
| 12 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换沙箱模式。仅在支持的平台上可用 |
| 13 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置 Claude Code 的状态栏。描述你想要的样式，或不带参数运行以从你的 shell prompt 自动配置 |
| 14 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 订购 Claude Code 贴纸 |
| 15 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置终端的 Shift+Enter 等快捷键。仅在需要它的终端中可见，如 VS Code、Cursor、Windsurf、Alacritty 或 Zed |
| 16 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 更改颜色主题。包括亮色和暗色变体、色盲友好（daltonized）主题、使用终端调色板的 ANSI 主题、跟随终端亮/暗模式的"自动（匹配终端）"选项，以及从 `~/.claude/themes/` 或插件加载的自定义主题。选择"新建自定义主题…"来创建自己的主题 |
| 17 | `/tui [default\|fullscreen]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置终端 UI 渲染方式并重新启动 Claude Code，保持当前对话不变。`default` 使用内联渲染；`fullscreen` 使用 alt-screen TUI |
| 18 | `/voice [hold\|tap\|off]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换语音听写，或在特定模式下启用。需要 Claude.ai 账户 |
| 19 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 以彩色网格可视化当前 Context 用量。显示对高 Context 工具、内存膨胀的优化建议和容量警告 |
| 20 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage` 的别名 |
| 21 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 配置超出限额后继续使用 |
| 22 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 生成分析报告，涵盖项目领域、交互模式和摩擦点 |
| 23 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage` 的别名。打开统计标签页 |
| 24 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 打开设置界面（状态标签页），显示版本、模型、账户和连接状态。在 Claude 回复期间也能使用，无需等待当前回复完成 |
| 25 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 显示会话费用、套餐用量限额和活动统计。`/cost` 和 `/stats` 是其别名 |
| 26 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 诊断和验证你的 Claude Code 安装和设置。结果显示为状态图标。按 `f` 让 Claude 修复报告的问题 |
| 27 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 提交关于 Claude Code 的反馈。别名：`/bug` |
| 28 | `/heapdump` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 将 JavaScript 堆快照和内存分解写入 `~/Desktop`，用于诊断高内存使用。在报告内存增长的 bug 时有用 |
| 29 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 显示帮助和可用命令 |
| 30 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 通过快速的交互式课程和动画演示发现 Claude Code 功能 |
| 31 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 在交互式版本选择器中查看变更日志。选择特定版本查看其发布说明，或选择显示所有版本 |
| 32 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 列出和管理后台任务。别名：`/bashes` |
| 33 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 复制最后一次助手回复到剪贴板。传入数字 `N` 可复制第 N 条最新回复：`/copy 2` 复制倒数第二条。当有代码块时，显示交互式选择器来选择单个代码块或完整回复。在选择器中按 `w` 将选择写入文件而非剪贴板，在 SSH 环境下很有用 |
| 34 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 将当前对话导出为纯文本。带文件名时直接写入该文件；不带时打开对话框复制到剪贴板或保存为文件 |
| 35 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 Agent 配置 |
| 36 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 配置 Chrome 中的 Claude 设置 |
| 37 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 查看工具事件的 Hook 配置 |
| 38 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 IDE 集成并显示状态 |
| 39 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 MCP 服务器连接和 OAuth 认证 |
| 40 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 Claude Code 插件 |
| 41 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 重新加载所有活动插件以应用待处理的更改，无需重启。报告每个重新加载的组件数量并标记加载错误 |
| 42 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 列出可用的 Skill。按 `t` 按 token 数量排序 |
| 43 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | 编辑 `CLAUDE.md` 记忆文件、启用或禁用自动记忆，以及查看自动记忆条目 |
| 44 | `/effort [low\|medium\|high\|xhigh\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 设置模型的努力级别。可用级别取决于模型，包括 `low`、`medium`、`high`、`xhigh` 和 `max`（仅会话级）。不带参数时打开交互式滑块选择级别。`auto` 重置为模型默认值。立即生效，无需等待当前回复完成 |
| 45 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 切换快速模式的开启或关闭 |
| 46 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 选择或更改 AI 模型。对于支持的模型，使用左右箭头调整努力级别。更改立即生效，无需等待当前回复完成。在对话中途切换模型且已有输出时，Claude 会在应用更改前发出警告 |
| 47 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 与朋友分享一周免费的 Claude Code。仅在你的账户符合条件时可见 |
| 48 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 直接从 prompt 进入 Plan Mode。可选传入描述以进入 Plan Mode 并立即开始该任务，例如 `/plan fix the auth bug` |
| 49 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 在 ultraplan 会话中起草计划，在浏览器中审查，然后远程执行或发送回终端 |
| 50 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 添加工作目录以在当前会话中访问文件。大多数 `.claude/` 配置不会从添加的目录中发现 |
| 51 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 打开交互式 diff 查看器，显示未提交的更改和每轮 diff。使用左右箭头在当前 git diff 和单个 Claude 轮次之间切换，使用上下箭头浏览文件 |
| 52 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 使用 `CLAUDE.md` 指南初始化项目。设置 `CLAUDE_CODE_NEW_INIT=1` 可进入交互式流程，还会引导配置 skills、hooks 和个人记忆文件 |
| 53 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 在当前会话中本地审查 pull request。如需更深入的基于云端的审查，请使用 `/ultrareview` |
| 54 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 分析当前分支上待处理的变更是否存在安全漏洞。审查 git diff 并识别注入、认证问题和数据泄露等风险 |
| 55 | `/team-onboarding` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 根据你的 Claude Code 使用历史生成团队入职指南。分析过去 30 天的会话、命令和 MCP 服务器使用情况 |
| 56 | `/ultrareview [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 在云端沙箱中对给定的 pull request 进行深度多 Agent 代码审查。生成带有优先级分类的结构化审查报告；补充本地 `/review` 命令 |
| 57 | `/autofix-pr [prompt]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在网页会话中生成一个 Claude Code，监控当前分支的 PR 并在 CI 失败或审查者留下评论时推送修复。通过 `gh pr view` 从你检出的分支检测开放的 PR；要监控不同的 PR，需先检出其分支。需要 `gh` CLI 和 Claude Code 网页版访问权限 |
| 58 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在 Claude Code 桌面应用中继续当前会话。仅限 macOS 和 Windows。别名：`/app` |
| 59 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 为仓库设置 Claude GitHub Actions 应用。引导你选择仓库并配置集成 |
| 60 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 安装 Claude Slack 应用。打开浏览器完成 OAuth 流程 |
| 61 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 显示二维码以下载 Claude 移动应用。别名：`/ios`、`/android` |
| 62 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使此会话可从 claude.ai 进行远程控制。别名：`/rc` |
| 63 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 为使用 `--remote` 启动的网页会话配置默认远程环境 |
| 64 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 创建、更新、列出或运行 routines。Claude 会以对话方式引导你完成设置。别名：`/routines` |
| 65 | `/teleport` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 将 Claude Code 网页会话拉取到此终端：打开选择器，然后获取分支和对话。也可用 `/tp`。需要 claude.ai 订阅 |
| 66 | `/web-setup` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使用本地 `gh` CLI 凭据将你的 GitHub 账户连接到 Claude Code 网页版。如果 GitHub 未连接，`/schedule` 会自动提示进行此设置 |
| 67 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 在此点创建当前对话的分支。别名：`/fork`。当设置了 `CLAUDE_CODE_FORK_SUBAGENT` 时，`/fork` 改为生成一个 forked subagent，不再为此命令的别名 |
| 68 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 快速提问一个旁注问题，不添加到对话中 |
| 69 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 以空 Context 开始新对话。之前的对话仍可在 `/resume` 中访问。要在继续同一对话时释放 Context，请使用 `/compact`。别名：`/reset`、`/new` |
| 70 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 压缩对话，可选附带聚焦指令 |
| 71 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 退出 CLI。别名：`/quit` |
| 72 | `/recap` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 按需生成当前会话的一行摘要，不影响正在进行的对话 |
| 73 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 重命名当前会话并在 prompt 栏上显示名称。不带名称时根据对话历史自动生成 |
| 74 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 通过 ID 或名称恢复对话，或打开会话选择器。别名：`/continue` |
| 75 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将对话和/或代码回退到之前的某个点，或从选定的消息开始总结。参见 checkpointing。别名：`/checkpoint`、`/undo` |

`/debug` 等捆绑的 Skill 也会出现在 slash-command 菜单中，但它们不是内置 Command。

---

## 来源

- [Claude Code Slash Commands（斜杠命令）](https://code.claude.com/docs/en/slash-commands)
- [Claude Code 交互模式](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code 变更日志](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

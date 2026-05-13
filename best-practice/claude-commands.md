# Commands 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-May%2012%2C%202026%2011%3A39%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.139-blue?style=flat&labelColor=555)<br>
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
| `name` | string | 否 | 显示名称和 `/slash-command` 标识符。省略时默认为目录名 |
| `description` | string | 推荐 | 命令的功能说明。显示在自动补全中，供 Claude 自动发现 |
| `when_to_use` | string | 否 | Claude 何时应该调用此命令的额外上下文——触发短语或示例请求。会追加到 `description` 中，计入 1,536 字符上限 |
| `argument-hint` | string | 否 | 自动补全时显示的提示（如 `[issue-number]`、`[filename]`） |
| `arguments` | string/list | 否 | 用于命令内容中 `$name` 替换的命名位置参数。接受空格分隔的字符串或 YAML 列表——名称按顺序映射到参数位置 |
| `disable-model-invocation` | boolean | 否 | 设为 `true` 阻止 Claude 自动调用此命令 |
| `user-invocable` | boolean | 否 | 设为 `false` 从 `/` 菜单中隐藏——命令仅作为背景知识 |
| `paths` | string/list | 否 | 限制此技能激活时机的 glob 模式。接受逗号分隔的字符串或 YAML 列表。设置后，Claude 仅在处理匹配模式的文件时自动加载此技能 |
| `allowed-tools` | string | 否 | 此命令激活时无需权限提示即可使用的工具 |
| `model` | string | 否 | 运行此命令时使用的模型（如 `haiku`、`sonnet`、`opus`） |
| `effort` | string | 否 | 调用时覆盖模型努力级别（`low`、`medium`、`high`、`max`） |
| `context` | string | 否 | 设为 `fork` 在隔离的子代理上下文中运行命令 |
| `agent` | string | 否 | 设置 `context: fork` 时的子代理类型（默认：`general-purpose`） |
| `shell` | string | 否 | `` !`command` `` 块使用的 shell——接受 `bash`（默认）或 `powershell`。需要设置 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | 否 | 作用域限定到此命令的生命周期钩子 |

---

## ![Official](../!/tags/official.svg) **(80)**

| # | Command | Tag | Description |
|---|---------|-----|-------------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 登录 Anthropic 账号 |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 退出 Anthropic 账号 |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Amazon Bedrock 认证、区域和模型绑定。仅在设置 `CLAUDE_CODE_USE_BEDROCK=1` 时可见。首次使用 Bedrock 的用户也可以从登录界面访问此向导 |
| 4 | `/setup-vertex` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Google Vertex AI 认证、项目、区域和模型绑定。仅在设置 `CLAUDE_CODE_USE_VERTEX=1` 时可见。首次使用 Vertex AI 的用户也可以从登录界面访问此向导 |
| 5 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 打开升级页面以切换到更高级别的套餐 |
| 6 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置当前会话的提示栏颜色。可用颜色：`red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink`、`cyan`。使用 `default` 重置 |
| 7 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开设置界面以调整主题、模型、输出风格等偏好设置。别名：`/settings` |
| 8 | `/focus` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换聚焦视图，仅显示最后一次提示、工具调用摘要和最终响应。有助于减少长会话中的视觉干扰。仅在全屏渲染中可用 |
| 9 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开或创建快捷键配置文件 |
| 10 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 管理工具权限的允许、询问和拒绝规则。打开交互式对话框，可以按范围查看规则、添加或删除规则、管理工作目录以及查看最近的 auto mode 拒绝记录。别名：`/allowed-tools` |
| 11 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 查看和更新隐私设置。仅适用于 Pro 和 Max 套餐订阅者 |
| 12 | `/radio` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 在浏览器中打开 Claude FM lo-fi 电台。无浏览器时打印流媒体 URL。不适用于 Bedrock、Vertex 或 Foundry |
| 13 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换沙盒模式。仅在支持的平台可用 |
| 14 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置 Claude Code 状态栏。描述你想要的效果，或不带参数运行以根据 shell 提示自动配置 |
| 15 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 订购 Claude Code 贴纸 |
| 16 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置 Shift+Enter 等终端快捷键。仅在需要它的终端中可见，如 VS Code、Cursor、Windsurf、Alacritty 或 Zed |
| 17 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 更改颜色主题。包括亮色和暗色变体、色盲友好（daltonized）主题、使用终端调色板的 ANSI 主题、跟随终端亮暗模式的 "Auto (match terminal)" 选项，以及从 `~/.claude/themes/` 或插件加载的自定义主题。选择 "New custom theme…" 创建自己的主题 |
| 18 | `/tui [default\|fullscreen]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置终端 UI 渲染器并重新启动 Claude Code 且保持当前对话不变。`default` 使用内联渲染；`fullscreen` 使用 alt-screen TUI |
| 19 | `/voice [hold\|tap\|off]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换语音听写，或以特定模式启用。需要 Claude.ai 账号 |
| 20 | `/context [all]` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 以彩色网格可视化当前上下文使用情况。显示对上下文密集型工具、内存膨胀和容量警告的优化建议。传入 `all` 在全屏中展开每项详细分解 |
| 21 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage` 的别名 |
| 22 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 配置额外用量以在达到速率限制时继续工作 |
| 23 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 生成分析报告，分析你的 Claude Code 会话，包括项目区域、交互模式和摩擦点 |
| 24 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage` 的别名。在 Stats 标签页打开 |
| 25 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 打开设置界面（Status 标签页），显示版本、模型、账号和连接状态。在 Claude 响应时也可使用，无需等待当前响应完成 |
| 26 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 显示会话费用、套餐用量限制和活动统计。`/cost` 和 `/stats` 是其别名 |
| 27 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 诊断并验证 Claude Code 安装和设置。结果显示状态图标。按 `f` 让 Claude 修复报告的问题 |
| 28 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 提交关于 Claude Code 的反馈。别名：`/bug` |
| 29 | `/heapdump` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 将 JavaScript 堆快照和内存使用分析写入 `~/Desktop`，用于诊断高内存使用。在提交内存增长 bug 报告时有用 |
| 30 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 显示帮助和可用命令 |
| 31 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 通过快速交互式课程和动画演示了解 Claude Code 功能 |
| 32 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 在交互式版本选择器中查看更新日志。选择特定版本查看其发布说明，或选择显示所有版本 |
| 33 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 列出和管理后台任务。别名：`/bashes` |
| 34 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 将最后一次助手响应复制到剪贴板。传入数字 `N` 复制倒数第 N 次响应：`/copy 2` 复制倒数第二次响应。存在代码块时，显示交互式选择器以选择单个代码块或完整响应。在选择器中按 `w` 可将选择内容写入文件而非剪贴板，在 SSH 环境下很有用 |
| 35 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 将当前对话导出为纯文本。带文件名时直接写入该文件。不带时打开对话框以复制到剪贴板或保存到文件 |
| 36 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理代理配置 |
| 37 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 在 Chrome 设置中配置 Claude |
| 38 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 查看工具事件的钩子配置 |
| 39 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 IDE 集成并显示状态 |
| 40 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 MCP 服务器连接和 OAuth 认证 |
| 41 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 Claude Code 插件 |
| 42 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 重新加载所有活动插件以应用待处理的更改，无需重启。报告每个重新加载组件的数量并标记任何加载错误 |
| 43 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 列出可用技能。按 `t` 按 token 数排序 |
| 44 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | 编辑 `CLAUDE.md` 记忆文件，启用或禁用自动记忆，以及查看自动记忆条目 |
| 45 | `/effort [low\|medium\|high\|xhigh\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 设置模型努力级别。可用级别取决于模型，包括 `low`、`medium`、`high`、`xhigh` 和 `max`（仅会话级）。不带参数时打开交互式滑块选择级别。`auto` 重置为模型默认值。立即生效，无需等待当前响应完成 |
| 46 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 切换快速模式开或关 |
| 47 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 选择或更改 AI 模型。对支持的模型，使用左右箭头调整努力级别。更改立即生效，无需等待当前响应完成。在已有输出的对话中途切换时，Claude 会在应用更改前发出警告 |
| 48 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 与朋友分享一周免费 Claude Code。仅在你的账号符合条件时可见 |
| 49 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 直接从提示进入计划模式。传入可选描述进入计划模式并立即开始该任务，例如 `/plan fix the auth bug` |
| 50 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 在 ultraplan 会话中起草计划，在浏览器中审查，然后远程执行或发回终端 |
| 51 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 添加当前会话的文件访问工作目录。大多数 `.claude/` 配置不会从添加的目录中发现 |
| 52 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 打开交互式 diff 查看器显示未提交的更改和每轮 diff。使用左右箭头在当前 git diff 和单个 Claude 轮次之间切换，上下浏览文件 |
| 53 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 使用 `CLAUDE.md` 指南初始化项目。设置 `CLAUDE_CODE_NEW_INIT=1` 进行交互式流程，还会遍历技能、钩子和个人记忆文件 |
| 54 | `/review [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 在当前会话中本地审查拉取请求。传入可选的 PR 编号或 URL 以定位特定 PR。更深入的云端审查，参见 `/ultrareview` |
| 55 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 分析当前分支待提交更改的安全漏洞。审查 git diff 并识别注入、认证问题和数据暴露等风险 |
| 56 | `/team-onboarding` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 根据你的 Claude Code 使用历史生成团队入职指南。分析过去 30 天的会话、命令和 MCP 服务器使用情况 |
| 57 | `/ultrareview [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 在云沙盒中对给定的拉取请求进行深度多代理代码审查。生成带有优先级发现的结构化审查报告；补充本地 `/review` 命令 |
| 58 | `/autofix-pr [prompt]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在网页端生成 Claude Code 会话来监视当前分支的 PR，在 CI 失败或审查者留下评论时推送修复。通过 `gh pr view` 从已检出的分支检测开放的 PR；要监视不同的 PR，先检出其分支。需要 `gh` CLI 和网页端 Claude Code 访问权限 |
| 59 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在 Claude Code Desktop 应用中继续当前会话。仅支持 macOS 和 Windows。别名：`/app` |
| 60 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 为仓库设置 Claude GitHub Actions 应用。引导你完成选择仓库和配置集成的流程 |
| 61 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 安装 Claude Slack 应用。打开浏览器完成 OAuth 流程 |
| 62 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 显示二维码以下载 Claude 移动应用。别名：`/ios`、`/android` |
| 63 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使此会话可从 claude.ai 进行远程控制。别名：`/rc` |
| 64 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 为使用 `--remote` 启动的网页会话配置默认远程环境 |
| 65 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 创建、更新、列出或运行例程。Claude 会以对话方式引导你完成设置。别名：`/routines` |
| 66 | `/teleport` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 将网页端的 Claude Code 会话拉入此终端：打开选择器，然后获取分支和对话。也可用作 `/tp`。需要 claude.ai 订阅 |
| 67 | `/web-setup` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使用本地 `gh` CLI 凭据将你的 GitHub 账号连接到网页端 Claude Code。如果 GitHub 未连接，`/schedule` 会自动提示进行此操作 |
| 68 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 在此点创建当前对话的分支。别名：`/fork`。设置 `CLAUDE_CODE_FORK_SUBAGENT` 时，`/fork` 改为生成分叉子代理，不再是此命令的别名 |
| 69 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 问一个快速附带问题，不添加到对话中 |
| 70 | `/clear [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 以空上下文开始新对话。传入可选名称在 `/resume` 选择器中标记之前的对话。之前的对话仍可在 `/resume` 中访问。要在继续同一对话的同时释放上下文，请改用 `/compact`。别名：`/reset`、`/new` |
| 71 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 压缩对话，可选聚焦指令 |
| 72 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 退出 CLI。别名：`/quit` |
| 73 | `/recap` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 按需生成当前会话的一行摘要，不影响正在进行的对话 |
| 74 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 重命名当前会话并在提示栏显示名称。不带名称时根据对话历史自动生成 |
| 75 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 通过 ID 或名称恢复对话，或打开会话选择器。别名：`/continue` |
| 76 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将对话和/或代码回退到之前的某个点，或从选定消息生成摘要。参见检查点机制。别名：`/checkpoint`、`/undo` |

`/debug` 等捆绑技能也可能出现在 slash 命令菜单中，但它们不是内置命令。

---

## 来源

- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

# Commands 最佳实践

<<<<<<< HEAD
![Last Updated](https://img.shields.io/badge/Last_Updated-Jun%2012%2C%202026%2011%3A08%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.175-blue?style=flat&labelColor=555)<br>
=======
![Last Updated](https://img.shields.io/badge/Last_Updated-Jun%2013%2C%202026%2011%3A07%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.176-blue?style=flat&labelColor=555)<br>
>>>>>>> upstream/main
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-commands-implementation.md)

Claude Code 命令 — frontmatter 字段和官方内置斜杠命令。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<<<<<<< HEAD
## Frontmatter 字段 (16)
=======
## Frontmatter Fields (16)
>>>>>>> upstream/main

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
<<<<<<< HEAD
| `name` | string | 否 | 显示名称和 `/斜杠命令` 标识符。如果省略则默认为目录名 |
| `description` | string | 推荐 | 命令的功能说明。在自动补全中显示，并被 Claude 用于自动发现 |
| `when_to_use` | string | 否 | Claude 何时调用此技能的附加上下文 — 触发短语或示例请求。追加到 `description` 后面显示在列表中，并计入 1,536 字符上限 |
| `argument-hint` | string | 否 | 自动补全时显示的提示（例如 `[issue-number]`、`[filename]`） |
| `arguments` | string/list | 否 | 用于命令内容中 `$name` 替换的命名位置参数。接受空格分隔的字符串或 YAML 列表 — 名称按顺序映射到参数位置 |
| `disable-model-invocation` | boolean | 否 | 设为 `true` 以阻止 Claude 自动调用此命令 |
| `user-invocable` | boolean | 否 | 设为 `false` 以从 `/` 菜单中隐藏 — 命令变为仅后台知识 |
| `paths` | string/list | 否 | 限制此技能激活时机的 glob 模式。接受逗号分隔的字符串或 YAML 列表。设置后，Claude 仅在处理与模式匹配的文件时才会自动加载该技能 |
| `allowed-tools` | string | 否 | 此命令激活时无需权限提示即可使用的工具 |
| `disallowed-tools` | string/list | 否 | 此命令激活时从 Claude 可用工具池中移除的工具。发送下一条消息时清除。与 `allowed-tools` 相反 |
| `model` | string | 否 | 运行此命令时使用的模型（例如 `haiku`、`sonnet`、`opus`） |
| `effort` | string | 否 | 调用时覆盖模型的努力级别（`low`、`medium`、`high`、`xhigh`、`max`） |
| `context` | string | 否 | 设为 `fork` 以在隔离的子代理上下文中运行命令 |
| `agent` | string | 否 | 当 `context: fork` 设置时的子代理类型（默认：`general-purpose`） |
| `shell` | string | 否 | `` !`command` `` 块使用的 shell — 接受 `bash`（默认）或 `powershell`。需要设置 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | 否 | 作用域限定到此命令的生命周期钩子 |
=======
| `name` | string | No | Display name and `/slash-command` identifier. Defaults to the directory name if omitted |
| `description` | string | Recommended | What the command does. Shown in autocomplete and used by Claude for auto-discovery |
| `when_to_use` | string | No | Additional context for when Claude should invoke the skill — trigger phrases or example requests. Appended to `description` in the listing and counts toward the 1,536-character cap |
| `argument-hint` | string | No | Hint shown during autocomplete (e.g., `[issue-number]`, `[filename]`) |
| `arguments` | string/list | No | Named positional arguments for `$name` substitution in command content. Accepts a space-separated string or YAML list — names map to argument positions in order |
| `disable-model-invocation` | boolean | No | Set `true` to prevent Claude from automatically invoking this command |
| `user-invocable` | boolean | No | Set `false` to hide from the `/` menu — command becomes background knowledge only |
| `paths` | string/list | No | Glob patterns that limit when this skill is activated. Accepts a comma-separated string or a YAML list. When set, Claude loads the skill automatically only when working with files matching the patterns |
| `allowed-tools` | string | No | Tools allowed without permission prompts when this command is active |
| `disallowed-tools` | string/list | No | Tools removed from Claude's available pool while this command is active. Clears when you send your next message. The inverse of `allowed-tools` |
| `model` | string | No | Model to use when this command runs (e.g., `haiku`, `sonnet`, `opus`) |
| `effort` | string | No | Override the model effort level when invoked (`low`, `medium`, `high`, `xhigh`, `max`) |
| `context` | string | No | Set to `fork` to run the command in an isolated subagent context |
| `agent` | string | No | Subagent type when `context: fork` is set (default: `general-purpose`) |
| `shell` | string | No | Shell for `` !`command` `` blocks — accepts `bash` (default) or `powershell`. Requires `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | No | Lifecycle hooks scoped to this command |
>>>>>>> upstream/main

---

## ![Official](../!/tags/official.svg) **(85)**

| # | 命令 | 标签 | 描述 |
|---|---------|-----|-------------|
<<<<<<< HEAD
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 登录你的 Anthropic 账号 |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 退出你的 Anthropic 账号 |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Amazon Bedrock 认证、区域和模型固定。仅在设置 `CLAUDE_CODE_USE_BEDROCK=1` 时可见。首次使用 Bedrock 的用户也可以从登录界面访问此向导 |
| 4 | `/setup-vertex` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Google Vertex AI 认证、项目、区域和模型固定。仅在设置 `CLAUDE_CODE_USE_VERTEX=1` 时可见。首次使用 Vertex AI 的用户也可以从登录界面访问此向导 |
| 5 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 打开升级页面以切换到更高级别的计划 |
| 6 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置当前会话的提示栏颜色。可用颜色：`red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink`、`cyan`。使用 `default` 重置。不带参数运行可随机选择颜色 |
| 7 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开设置界面以调整主题、模型、输出风格和其他偏好设置。别名：`/settings` |
| 8 | `/focus` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换聚焦视图，仅显示最近的提示、工具调用摘要和最终回复。有助于在长时间会话中减少视觉干扰。仅在全屏渲染模式下可用 |
| 9 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开或创建你的按键绑定配置文件 |
| 10 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 管理工具权限的允许、询问和拒绝规则。打开交互式对话框，你可以按作用域查看规则、添加或删除规则、管理工作目录以及查看最近的自动模式拒绝记录。别名：`/allowed-tools` |
| 11 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 查看和更新你的隐私设置。仅对 Pro 和 Max 计划订阅者可用 |
| 12 | `/radio` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 在浏览器中打开 Claude FM lo-fi 电台 |
| 13 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换沙盒模式。仅在支持的平台上可用 |
| 14 | `/scroll-speed` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 交互式调整鼠标滚轮滚动速度 |
| 15 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置 Claude Code 的状态栏。描述你想要的效果，或不带参数运行以根据 shell 提示自动配置 |
| 16 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 订购 Claude Code 贴纸 |
| 17 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置终端按键绑定，用于 Shift+Enter 和其他快捷键。仅在需要它的终端中可见，如 VS Code、Cursor、Devin Desktop、Alacritty 或 Zed |
| 18 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 更改颜色主题。包括明暗变体、色盲友好（去色）主题、使用终端调色板的 ANSI 主题、跟随终端明暗模式的"自动（匹配终端）"选项，以及从 `~/.claude/themes/` 或插件加载的自定义主题。选择"新建自定义主题…"来创建你自己的主题 |
| 19 | `/tui [default\|fullscreen]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置终端 UI 渲染器并使用当前对话重新启动 Claude Code。`default` 使用内联渲染；`fullscreen` 使用替代屏幕 TUI |
| 20 | `/voice [hold\|tap\|off]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换语音输入，或以特定模式启用。需要 Claude.ai 账号 |
| 21 | `/context [all]` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 将当前上下文使用情况可视化为彩色网格。显示对上下文密集工具、内存膨胀的优化建议和容量警告。传入 `all` 展开完整详情 |
| 22 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage` 的别名 |
| 23 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 生成分析报告，涵盖 Claude Code 会话的项目领域、交互模式和摩擦点 |
| 24 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage` 的别名。在 Stats 标签页打开 |
| 25 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 打开设置界面（Status 标签页），显示版本、模型、账号和连接状态。在 Claude 正在响应时也可使用，无需等待当前回复完成 |
| 26 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 显示会话费用、计划用量限制和活动统计。`/cost` 和 `/stats` 是其别名 |
| 27 | `/usage-credits` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 配置用量额度以在达到限制时继续工作。原名为 `/extra-usage` |
| 28 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 诊断并验证你的 Claude Code 安装和设置。结果以状态图标显示。按 `f` 让 Claude 修复报告的任何问题 |
| 29 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 提交反馈、报告 bug 或分享你的对话。别名：`/bug`、`/share` |
| 30 | `/heapdump` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 将 JavaScript 堆快照和内存使用情况写入 `~/Desktop`，用于诊断高内存占用。在报告内存增长的 bug 时很有用 |
| 31 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 显示帮助和可用命令 |
| 32 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 通过带有动画演示的快速互动课程发现 Claude Code 功能 |
| 33 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 在交互式版本选择器中查看变更日志。选择特定版本查看其发行说明，或选择显示所有版本 |
| 34 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 列出和管理后台任务。别名：`/bashes` |
| 35 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 将最近的助手回复复制到剪贴板。传入数字 `N` 可复制倒数第 N 条回复：`/copy 2` 复制倒数第二条。当存在代码块时，显示交互式选择器以选择单个代码块或完整回复。在选择器中按 `w` 将选中内容写入文件而非剪贴板，这在 SSH 环境下很有用 |
| 36 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 将当前对话导出为纯文本。带文件名时直接写入该文件；不带时打开对话框以复制到剪贴板或保存到文件 |
| 37 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理代理配置 |
| 38 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 在 Chrome 设置中配置 Claude |
| 39 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 查看工具事件的钩子配置 |
| 40 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 IDE 集成并显示状态 |
| 41 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 MCP 服务器连接和 OAuth 认证 |
| 42 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 Claude Code 插件 |
| 43 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 重新加载所有已激活的插件以应用待处理的更改，无需重新启动。报告每个重新加载组件的计数并标记任何加载错误 |
| 44 | `/reload-skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 重新扫描技能和命令目录，使会话期间在磁盘上添加或更改的技能无需重启即可生效。报告可用技能数量以及新增或移除的数量 |
| 45 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 列出可用技能。按 `t` 按 token 数排序 |
| 46 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | 编辑 `CLAUDE.md` 记忆文件、启用或禁用自动记忆，以及查看自动记忆条目 |
| 47 | `/advisor [model|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 启用或禁用 advisor 工具，该工具在任务的关键时刻咨询第二个模型以获取指导。接受模型名称（`opus`、`sonnet`、`fable`）或完整模型 ID；不带参数时打开选择器。使用 `off` 禁用 |
| 48 | `/effort [low\|medium\|high\|xhigh\|max\|ultracode]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 设置模型的努力级别。可用级别取决于所选模型，包括 `low`、`medium`、`high`、`xhigh`、`max`（仅当前会话）和 `ultracode`（结合 `xhigh` 推理与自动工作流编排；仅当前会话）。不带参数时打开交互式滑块来选择级别。`auto` 重置为模型默认值。立即生效，无需等待当前回复完成 |
| 49 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 开启或关闭快速模式 |
| 50 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 选择或更改 AI 模型。对于支持的模型，使用左右箭头调整努力级别。更改立即生效，无需等待当前回复完成。在已有输出的对话中途切换时，Claude 会在应用更改前发出警告 |
| 51 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 与朋友分享一周免费的 Claude Code 使用权限。仅在你的账号符合条件时可见 |
| 52 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 直接从提示进入计划模式。可选传入描述以进入计划模式并立即开始该任务，例如 `/plan fix the auth bug` |
| 53 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 在 ultraplan 会话中起草计划，在浏览器中审阅，然后远程执行或发送回终端 |
| 54 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 为当前会话添加工作目录以进行文件访问。大多数 `.claude/` 配置不会从添加的目录中发现 |
| 55 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 打开交互式 diff 查看器，显示未提交的更改和每轮 diff。使用左右箭头在当前 git diff 和各个 Claude 轮次之间切换，使用上下方向键浏览文件 |
| 56 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 使用 `CLAUDE.md` 指南初始化项目。设置 `CLAUDE_CODE_NEW_INIT=1` 可进行交互式流程，同时引导配置技能、钩子和个人记忆文件 |
| 57 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 在当前会话中本地审阅 pull request。如需更深入的云端审阅，请参阅 `/ultrareview` |
| 58 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 分析当前分支上的待处理更改中的安全漏洞。审查 git diff 并识别注入、认证问题和数据泄露等风险 |
| 59 | `/team-onboarding` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 从你的 Claude Code 使用历史中生成团队入职指南。分析过去 30 天的会话、命令和 MCP 服务器使用情况 |
| 60 | `/ultrareview [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 在云端沙箱中对指定的 pull request 进行深度多代理代码审查。生成带有优先级排序的结构化审查结果；是对本地 `/review` 命令的补充 |
| 61 | `/autofix-pr [prompt]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在 web 会话中启动一个 Claude Code 来监视当前分支的 PR，并在 CI 失败或审查者留下评论时推送修复。通过 `gh pr view` 从你检出的分支检测开放的 PR；要监视不同的 PR，先检出该分支。需要 `gh` CLI 和 Claude Code web 版的访问权限 |
| 62 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在 Claude Code 桌面应用中继续当前会话。仅支持 macOS 和 Windows。别名：`/app` |
| 63 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 为仓库设置 Claude GitHub Actions 应用。引导你选择仓库并配置集成 |
| 64 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 安装 Claude Slack 应用。打开浏览器以完成 OAuth 流程 |
| 65 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 显示二维码以下载 Claude 移动应用。别名：`/ios`、`/android` |
| 66 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使此会话可从 claude.ai 进行远程控制。别名：`/rc` |
| 67 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 选择云端代理的默认环境 |
| 68 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 创建、更新、列出或运行例行任务。Claude 会通过交互式对话引导你完成设置。别名：`/routines` |
| 69 | `/teleport` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 将 web 会话中的 Claude Code 拉取到当前终端：打开选择器，然后获取分支和对话。也可用 `/tp`。需要 claude.ai 订阅 |
| 70 | `/web-setup` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使用本地 `gh` CLI 凭据将你的 GitHub 账号连接到 Claude Code web 版。如果 GitHub 未连接，`/schedule` 会自动提示进行此操作 |
| 71 | `/background [prompt]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将当前会话分离为后台代理运行，释放此终端。别名：`/bg` |
| 72 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 在此点创建当前对话的分支 |
| 73 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 快速提问一个附带问题，不会添加到主对话中 |
| 74 | `/cd <path>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将工作目录切换到新路径，而不会破坏提示缓存 |
| 75 | `/clear [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 开始一个清空上下文的新对话。传入可选的 `name` 为之前的对话命名，方便通过 `/resume` 检索。要在继续同一对话的同时释放上下文，请使用 `/compact`。别名：`/reset`、`/new` |
| 76 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 压缩对话，可附带聚焦指令 |
| 77 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 退出 CLI。别名：`/quit` |
| 78 | `/fork <directive>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 启动一个分叉的子代理：一个继承完整对话并在后台按照指令工作的子代理，同时你继续进行操作 |
| 79 | `/goal [condition\|clear]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 设置一个目标 — Claude 会持续工作直到条件满足。传入 `clear` 以移除现有目标 |
| 80 | `/recap` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 按需生成当前会话的一行摘要，不影响正在进行的对话 |
| 81 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 重命名当前会话并在提示栏显示名称。不带名称时，根据对话历史自动生成一个 |
| 82 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 通过 ID 或名称恢复一个对话，或打开会话选择器。别名：`/continue` |
| 83 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将对话和/或代码回退到之前的某个点，或从选定的消息开始摘要。参见检查点功能。别名：`/checkpoint`、`/undo` |
| 84 | `/stop` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 停止当前后台会话。保留工作区和对话记录 |
| 85 | `/workflows` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 打开工作流进度视图，以监视、暂停、恢复或保存正在运行和已完成的工作流 |
=======
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Sign in to your Anthropic account |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Sign out from your Anthropic account |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Configure Amazon Bedrock authentication, region, and model pins through an interactive wizard. Only visible when `CLAUDE_CODE_USE_BEDROCK=1` is set. First-time Bedrock users can also access this wizard from the login screen |
| 4 | `/setup-vertex` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Configure Google Vertex AI authentication, project, region, and model pins through an interactive wizard. Only visible when `CLAUDE_CODE_USE_VERTEX=1` is set. First-time Vertex AI users can also access this wizard from the login screen |
| 5 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Open the upgrade page to switch to a higher plan tier |
| 6 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Set the prompt bar color for the current session. Available colors: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. Use `default` to reset. Run without an argument to pick a random color |
| 7 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Open the Settings interface to adjust theme, model, output style, and other preferences. Alias: `/settings` |
| 8 | `/focus` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Toggle the focus view, which shows only the last prompt, a summary of tool calls, and the final response. Useful for reducing visual noise during long sessions. Only available in fullscreen rendering |
| 9 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Open or create your keybindings configuration file |
| 10 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Manage allow, ask, and deny rules for tool permissions. Opens an interactive dialog where you can view rules by scope, add or remove rules, manage working directories, and review recent auto mode denials. Alias: `/allowed-tools` |
| 11 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | View and update your privacy settings. Only available for Pro and Max plan subscribers |
| 12 | `/radio` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Open Claude FM lo-fi radio in your browser |
| 13 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Toggle sandbox mode. Available on supported platforms only |
| 14 | `/scroll-speed` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Adjust mouse wheel scroll speed interactively |
| 15 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Configure Claude Code's status line. Describe what you want, or run without arguments to auto-configure from your shell prompt |
| 16 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Order Claude Code stickers |
| 17 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Configure terminal keybindings for Shift+Enter and other shortcuts. Only visible in terminals that need it, like VS Code, Cursor, Devin Desktop, Alacritty, or Zed |
| 18 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Change the color theme. Includes light and dark variants, colorblind-accessible (daltonized) themes, ANSI themes that use your terminal's color palette, an "Auto (match terminal)" option that follows your terminal's light/dark mode, and custom themes loaded from `~/.claude/themes/` or plugins. Select "New custom theme…" to create your own |
| 19 | `/tui [default\|fullscreen]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Set the terminal UI renderer and relaunch Claude Code with the current conversation intact. `default` uses inline rendering; `fullscreen` uses an alt-screen TUI |
| 20 | `/voice [hold\|tap\|off]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Toggle voice dictation, or enable it in a specific mode. Requires a Claude.ai account |
| 21 | `/context [all]` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Visualize current context usage as a colored grid. Shows optimization suggestions for context-heavy tools, memory bloat, and capacity warnings. Pass `all` to expand the full breakdown |
| 22 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Alias for `/usage` |
| 23 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Generate a report analyzing your Claude Code sessions, including project areas, interaction patterns, and friction points |
| 24 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Alias for `/usage`. Opens on the Stats tab |
| 25 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Open the Settings interface (Status tab) showing version, model, account, and connectivity. Works while Claude is responding, without waiting for the current response to finish |
| 26 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Show session cost, plan usage limits, and activity stats. `/cost` and `/stats` are aliases |
| 27 | `/usage-credits` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Configure usage credits to keep working when you hit a limit. Previously `/extra-usage` |
| 28 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Diagnose and verify your Claude Code installation and settings. Results show with status icons. Press `f` to have Claude fix any reported issues |
| 29 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Submit feedback, report a bug, or share your conversation. Aliases: `/bug`, `/share` |
| 30 | `/heapdump` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Write a JavaScript heap snapshot and memory breakdown to `~/Desktop` for diagnosing high memory usage. Useful when filing bug reports about memory growth |
| 31 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Show help and available commands |
| 32 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Discover Claude Code features through quick interactive lessons with animated demos |
| 33 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | View the changelog in an interactive version picker. Select a specific version to see its release notes, or choose to show all versions |
| 34 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | List and manage background tasks. Alias: `/bashes` |
| 35 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | Copy the last assistant response to clipboard. Pass a number `N` to copy the Nth-latest response: `/copy 2` copies the second-to-last. When code blocks are present, shows an interactive picker to select individual blocks or the full response. Press `w` in the picker to write the selection to a file instead of the clipboard, which is useful over SSH |
| 36 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | Export the current conversation as plain text. With a filename, writes directly to that file. Without, opens a dialog to copy to clipboard or save to a file |
| 37 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Manage agent configurations |
| 38 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Configure Claude in Chrome settings |
| 39 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | View hook configurations for tool events |
| 40 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Manage IDE integrations and show status |
| 41 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Manage MCP server connections and OAuth authentication |
| 42 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Manage Claude Code plugins |
| 43 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Reload all active plugins to apply pending changes without restarting. Reports counts for each reloaded component and flags any load errors |
| 44 | `/reload-skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Re-scan skill and command directories so skills added or changed on disk during the session become available without restarting. Reports how many skills are available and how many were added or removed |
| 45 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | List available skills. Press `t` to sort by token count |
| 46 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | Edit `CLAUDE.md` memory files, enable or disable auto-memory, and view auto-memory entries |
| 47 | `/advisor [model\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Enable or disable the advisor tool, which consults a second model for guidance at key moments during a task. Accepts a model name (`opus`, `sonnet`, `fable`) or a full model ID; without an argument opens a picker. Use `off` to disable |
| 48 | `/effort [low\|medium\|high\|xhigh\|max\|ultracode]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Set the model effort level. Available levels depend on the model and include `low`, `medium`, `high`, `xhigh`, `max` (session-only), and `ultracode` (combines `xhigh` reasoning with automatic workflow orchestration; session-only). Without an argument, opens an interactive slider to pick the level. `auto` resets to the model default. Takes effect immediately without waiting for the current response to finish |
| 49 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Toggle fast mode on or off |
| 50 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Select or change the AI model. For models that support it, use left/right arrows to adjust effort level. The change takes effect immediately without waiting for the current response to finish. When switching mid-conversation after prior output, Claude warns before applying the change |
| 51 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Share a free week of Claude Code with friends. Only visible if your account is eligible |
| 52 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Enter plan mode directly from the prompt. Pass an optional description to enter plan mode and immediately start with that task, for example `/plan fix the auth bug` |
| 53 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Draft a plan in an ultraplan session, review it in your browser, then execute remotely or send it back to your terminal |
| 54 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Add a working directory for file access during the current session. Most `.claude/` configuration is not discovered from the added directory |
| 55 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Open an interactive diff viewer showing uncommitted changes and per-turn diffs. Use left/right arrows to switch between the current git diff and individual Claude turns, and up/down to browse files |
| 56 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Initialize project with a `CLAUDE.md` guide. Set `CLAUDE_CODE_NEW_INIT=1` for an interactive flow that also walks through skills, hooks, and personal memory files |
| 57 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Review a pull request locally in your current session. For a deeper cloud-based review, see `/code-review ultra` |
| 58 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Analyze pending changes on the current branch for security vulnerabilities. Reviews the git diff and identifies risks like injection, auth issues, and data exposure |
| 59 | `/team-onboarding` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Generate a team onboarding guide from your Claude Code usage history. Analyzes sessions, commands, and MCP server usage from the past 30 days |
| 60 | `/ultrareview [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Run a deep, multi-agent code review of the given pull request in a cloud sandbox. Produces a structured review with prioritized findings. The preferred invocation is `/code-review ultra`; `/ultrareview` remains as an alias |
| 61 | `/autofix-pr [prompt]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Spawn a Claude Code on the web session that watches the current branch's PR and pushes fixes when CI fails or reviewers leave comments. Detects the open PR from your checked-out branch with `gh pr view`; to watch a different PR, check out its branch first. Requires the `gh` CLI and access to Claude Code on the web |
| 62 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Continue the current session in the Claude Code Desktop app. macOS and Windows only. Alias: `/app` |
| 63 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Set up the Claude GitHub Actions app for a repository. Walks you through selecting a repo and configuring the integration |
| 64 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Install the Claude Slack app. Opens a browser to complete the OAuth flow |
| 65 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Show QR code to download the Claude mobile app. Aliases: `/ios`, `/android` |
| 66 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Make this session available for remote control from claude.ai. Alias: `/rc` |
| 67 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Choose the default environment for cloud agents |
| 68 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Create, update, list, or run routines. Claude walks you through the setup conversationally. Alias: `/routines` |
| 69 | `/teleport` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Pull a Claude Code on the web session into this terminal: opens a picker, then fetches the branch and conversation. Also available as `/tp`. Requires a claude.ai subscription |
| 70 | `/web-setup` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Connect your GitHub account to Claude Code on the web using your local `gh` CLI credentials. `/schedule` prompts for this automatically if GitHub is not connected |
| 71 | `/background [prompt]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Detach the current session to run as a background agent and free this terminal. Alias: `/bg` |
| 72 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Create a branch of the current conversation at this point |
| 73 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Ask a quick side question without adding to the conversation |
| 74 | `/cd <path>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Move the session to a new working directory without breaking the prompt cache |
| 75 | `/clear [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Start a new conversation with empty context. Pass an optional `name` to label the previous conversation for easy retrieval via `/resume`. To free up context while continuing the same conversation, use `/compact` instead. Aliases: `/reset`, `/new` |
| 76 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Compact conversation with optional focus instructions |
| 77 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Exit the CLI. Alias: `/quit` |
| 78 | `/fork <directive>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Spawn a forked subagent: a background subagent that inherits the full conversation and works on the directive while you keep going |
| 79 | `/goal [condition\|clear]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Set a goal — Claude keeps working across turns until the condition is met. Pass `clear` to remove an existing goal |
| 80 | `/recap` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Generate a one-line summary of the current session on demand, without affecting the ongoing conversation |
| 81 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Rename the current session and show the name on the prompt bar. Without a name, auto-generates one from conversation history |
| 82 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Resume a conversation by ID or name, or open the session picker. Alias: `/continue` |
| 83 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Rewind the conversation and/or code to a previous point, or summarize from a selected message. See checkpointing. Alias: `/checkpoint`, `/undo` |
| 84 | `/stop` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Stop the current background session. The transcript and worktree are kept |
| 85 | `/workflows` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Open the workflow progress view to watch, pause, resume, or save running and completed workflows |
>>>>>>> upstream/main

捆绑技能如 `/debug` 也可能出现在斜杠命令菜单中，但它们不是内置命令。

---

## 来源

- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

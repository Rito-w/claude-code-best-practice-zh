# Commands 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Jun%2011%2C%202026%2011%3A09%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.173-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-commands-implementation.md)

Claude Code 命令 — frontmatter 字段和官方内置斜杠命令。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter 字段 (16)

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
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

---

## ![Official](../!/tags/official.svg) **(85)**

| # | 命令 | 标签 | 描述 |
|---|---------|-----|-------------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 登录你的 Anthropic 账号 |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 退出你的 Anthropic 账号 |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Amazon Bedrock 认证、区域和模型固定。仅在设置 `CLAUDE_CODE_USE_BEDROCK=1` 时可见。首次使用 Bedrock 的用户也可以从登录界面访问此向导 |
| 4 | `/setup-vertex` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Google Vertex AI 认证、项目、区域和模型固定。仅在设置 `CLAUDE_CODE_USE_VERTEX=1` 时可见。首次使用 Vertex AI 的用户也可以从登录界面访问此向导 |
| 5 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 打开升级页面以切换到更高级别的计划 |
| 6 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置当前会话的提示栏颜色。可用颜色：`red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink`、`cyan`。使用 `default` 重置 |
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
| 21 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 将当前上下文使用情况可视化为彩色网格。显示对上下文密集工具、内存膨胀的优化建议和容量警告 |
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
| 67 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 配置使用 `--remote` 启动的 web 会话的默认远程环境 |
| 68 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 创建、更新、列出或运行例行任务。Claude 会通过交互式对话引导你完成设置。别名：`/routines` |
| 69 | `/teleport` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 将 web 会话中的 Claude Code 拉取到当前终端：打开选择器，然后获取分支和对话。也可用 `/tp`。需要 claude.ai 订阅 |
| 70 | `/web-setup` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使用本地 `gh` CLI 凭据将你的 GitHub 账号连接到 Claude Code web 版。如果 GitHub 未连接，`/schedule` 会自动提示进行此操作 |
| 71 | `/background [prompt]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将当前会话分离为后台代理运行，释放此终端。别名：`/bg` |
| 72 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 在此点创建当前对话的分支 |
| 73 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 快速提问一个附带问题，不会添加到主对话中 |
| 74 | `/cd [path]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将工作目录切换到新路径，而不会破坏提示缓存 |
| 75 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 开始一个清空上下文的新对话。之前的对话仍可在 `/resume` 中访问。要在继续同一对话的同时释放上下文，请使用 `/compact`。别名：`/reset`、`/new` |
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

捆绑技能如 `/debug` 也可能出现在斜杠命令菜单中，但它们不是内置命令。

---

## 来源

- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

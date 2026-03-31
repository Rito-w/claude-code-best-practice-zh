<!-- 翻译标记：agent-teams/agent-teams-prompt.md - 已翻译 -->
创建一个 Agent Team 来构建时间编排工作流，将当前迪拜时间显示为视觉 SVG 卡片。工作流遵循 **Command → Agent → Skill** 架构模式：

- Command 编排流程并处理用户交互
- Agent 使用预加载的 Skill 获取迪拜的实时当前时间
- Skill 从获取的时间数据创建视觉 SVG 时间卡片

**重要**：所有文件必须创建在 `agent-teams/.claude/` 中 — 而不是仓库根目录的 `.claude/` 目录中。这使 Agent Team 的输出自包含且可通过 `cd agent-teams && claude` 运行。不要引用或复制现有的 Weather 工作流 — 从头开始构建所有内容。

分配这些 Teammates：

1. **Command Architect** — 在 `agent-teams/.claude/commands/time-orchestrator.md` 中设计和实现 `/time-orchestrator` Command。Command 应该：
   - 通过 Agent 工具（不是 Bash）调用 time-agent 获取迪拜，阿联酋的当前时间（Asia/Dubai 时区，UTC+4）
   - 通过 Skill 工具调用 time-svg-creator Skill 从获取的时间数据渲染 SVG 卡片
   - 在 Frontmatter 中使用 model: haiku
   - 包括关键要求：顺序流程，正确的工具使用（Agents 用 Agent 工具，Skills 用 Skill 工具）和输出摘要
   通过共享任务列表与其他 Teammates 协调，就组件之间传递的数据契约（{time, timezone, formatted}）达成一致。

2. **Agent Engineer** — 在 `agent-teams/.claude/agents/time-agent.md` 中设计和实现 `time-agent`，在其预加载的 `time-fetcher` Skill 在 `agent-teams/.claude/skills/time-fetcher/SKILL.md` 中。Agent 应该：
   - 使用 Bash 与 `TZ='Asia/Dubai' date '+%Y-%m-%d %H:%M:%S %Z'` 获取迪拜的当前时间（Asia/Dubai, UTC+4）
   - 向 Command 返回时间值、时区名称和格式化字符串
   - 使用 Frontmatter：tools (Bash), model: haiku, color: blue, maxTurns: 3
   - 通过 `skills:` 字段预加载 time-fetcher Skill
   time-fetcher Skill（`agent-teams/.claude/skills/time-fetcher/SKILL.md`）应该包含迪拜时间的 Bash 命令、预期输出格式，并设置 user-invocable: false 因为它是仅 Agent 的领域知识。将商定的数据契约发布到共享任务列表，以便 Command Architect 和 Skill Designer 可以对齐接口。

3. **Skill Designer** — 在 `agent-teams/.claude/skills/time-svg-creator/SKILL.md` 中设计和实现 `time-svg-creator` Skill，带有支持文件 `reference.md`（SVG 模板 + 输出模板）和 `examples.md`（示例输入/输出对）。Skill 应该：
   - 从调用 Context 接收时间值、时区和格式化字符串
   - 为迪拜创建自包含的 SVG 时间卡片显示当前时间
   - 将 SVG 写入 `agent-teams/output/dubai-time.svg`
   - 将 Markdown 摘要写入 `agent-teams/output/output.md`
   - 使用提供的确切时间 — 从不重新获取
   - 将模板保留在 reference.md 中（带有占位符的 SVG 标记，Markdown 输出模板），示例对保留在 examples.md 中
   还创建 `agent-teams/output/` 目录用于输出文件。

所有三个 Teammates 应该在共享任务列表中创建任务以协调数据契约：Agent 返回 {time, timezone, formatted}，Command 通过 Context 传递它，Skill 消费它。并行启动所有三个，因为组件是独立的 — 它们只需要就数据接口达成一致，不需要等待彼此的实现。

---
name: vibe-to-agentic-framework
description: Presentation 背后的概念框架 — "Vibe Coding to Agentic Engineering" 的含义、为什么旅程如此结构化，以及每个幻灯片如何适应叙事弧线
---

<!-- 翻译标记：.claude/skills/presentation/vibe-to-agentic-framework/SKILL.md - 已翻译 -->

# "Vibe Coding to Agentic Engineering" 框架

此 Skill 教授 Presentation 背后的 **概念模型**。每个幻灯片和章节都存在是为了讲述一个故事：开发者如何逐步从无结构的 "vibe coding"（Low 级别）移动到高级 agentic engineering（High 级别）。

## 核心概念

**Vibe Coding（Low 级别）** 是开发者使用 Claude Code 时没有任何结构 — 没有项目上下文、没有约定、没有可重用的知识。每个提示都是抛硬币。Claude 可能会创建随机的端点、忽略现有模式、跳过测试，并产生不一致的代码。代码库随着每次交互向熵漂移。

**Agentic Engineering（High 级别）** 是 Claude Code 作为完全配置的工程系统运行。它了解项目架构（CLAUDE.md）、遵循范围限定的约定（Rules）、按需加载领域专业知识（Skills）、委托给专门的工作者（Agents）、编排多步骤工作流（Commands）、自动化生命周期事件（Hooks），并连接到外部工具（MCP Servers）。每个提示都产生一致的、经过测试的、生产质量的代码。

这两个极端之间的旅程是 **渐进和累积的**。每个最佳实践都建立在前一个之上，Presentation 按开发者应该采用的顺序教授它们。

## 4 级旅程系统

Presentation 使用 4 级评分系统而不是百分比条：

| 级别 | 顺序 | 颜色 | 旅程条高度 | 描述 |
|-------|-------|-------|--------------------|-------------|
| Low | 1 | 红/橙色 (`hsl(0, 70%, 45%)`) | 25% | Vibe coding 领域 — 无结构 |
| Medium | 2 | 黄色 (`hsl(40, 70%, 45%)`) | 50% | 结构化工作流，一些自动化 |
| High | 3 | 浅绿色 (`hsl(80, 70%, 45%)`) | 75% | 领域知识、skills、自定义 agents |
| Pro | 4 | 深绿色 (`hsl(120, 70%, 45%)`) | 100% | 完整 agentic engineering，多 agent 团队 |

旅程条在幻灯片 1（标题幻灯片）上隐藏，从幻灯片 2 开始显示。级别通过关键转换幻灯片上的 `data-level` 属性设置，并由后续幻灯片继承直到下一个级别更改。当级别更改时，`.level-badge` 会在幻灯片的 `h1` 上通过 JS 注入（不要在 HTML 中硬编码这些）。

## 运行示例：TodoApp Monorepo

每个技术都在现实的全栈项目上演示。Presentation 展示了从普通项目（vibe coding）到具有完整 Claude Code 配置的项目（agentic engineering）的转换：

**之前（Vibe Coding）：**
```
todoapp/
├── backend/          # FastAPI (Python)
│   ├── main.py
│   ├── routes/
│   ├── models/
│   └── tests/
└── frontend/         # Next.js (TypeScript)
    ├── components/
    ├── pages/
    └── lib/
```

**之后（Agentic Engineering）：**
```
todoapp/
├── .claude/                  # Claude Code 配置
│   ├── agents/               # 自定义 subagents
│   ├── skills/               # 领域知识
│   ├── commands/             # Slash commands
│   ├── hooks/                # 生命周期脚本
│   ├── rules/                # 模块化指令
│   ├── settings.json         # 团队设置
│   └── settings.local.json   # 个人设置
├── backend/
│   └── CLAUDE.md             # 后端指令
├── frontend/
│   └── CLAUDE.md             # 前端指令
├── .mcp.json                 # 托管 MCP servers
└── CLAUDE.md                 # 项目指令
```

**为什么是 TodoApp？** 它足够小可以放在幻灯片上，但又足够复杂可以演示真实问题：具有路由模式和测试约定的后端、具有组件层次结构和设计令牌的前端，以及需要两侧协调的跨领域关注点（如添加新功能）的 monorepo 结构。

TodoApp 使 vibe-coding 问题具体化：没有结构的情况下，要求 Claude "添加 notes 功能" 会产生一个不遵循 `routes/todos.py` 模式的随机 `/api/notes` 端点、一个没有侧边栏导航的独立页面，以及零测试。使用完整的 agentic 设置，相同的请求会产生遵循现有模式的路由、集成到侧边栏的页面，以及匹配 `test_todos.py` 风格的测试。

## 旅程弧线：为什么是这个顺序

Presentation 遵循有意的教学顺序。每个章节解锁一个新的能力层：

### Part 0: 介绍（幻灯片 1–4，无权重）
**目的：** 设定舞台。介绍 TodoApp，定义 vibe coding，并展示目的地。
- 标题幻灯片建立旅程隐喻
- 示例项目展示转换：TodoApp 的前后对比 — 普通项目结构与具有完整 Claude Code 配置的项目（.claude/、CLAUDE.md、.mcp.json 等）
- "什么是 Vibe Coding？" 创建 0% 基线 — 痛点
- 旅程地图提供可点击的 TOC，显示前方的完整路径

### Part 1: 先决条件（幻灯片 5–9，无权重）
**目的：** 安装并运行 Claude Code。这纯粹是后勤工作 — 还没有工程实践。
- 安装、认证、第一次会话、界面概述
- 无权重因为知道如何安装工具不会提高代码质量
- "第一次会话" 就是 vibe coding — 这是有意的，所以开发者亲身体验 0% 状态

### Part 2: 更好的提示（幻灯片 10–17，级别：Low）
**目的：** 第一个真正的改进。更好的输入产生更好的输出，即使没有任何项目配置。
- **好与坏的提示：** 具体的、范围限定的提示与模糊的请求。最简单的改进。
- **提供上下文：** 使用 `@files` 给 Claude 它需要的代码。立即减少幻觉。
- **上下文窗口和 /compact：** 理解有限的上下文窗口防止长会话中响应降级。
- **Plan Mode:** `/plan` 强制在编码前思考。防止在错误的方法上浪费精力。

**为什么是 Low 级别：** 提示是基础但有限的。它改进单个交互但不创建持久的项目知识。每个会话从零开始。

### Part 3: 项目记忆（幻灯片 18–24，级别：Medium）
**目的：** 从会话级到项目级知识的飞跃。Claude 现在跨会话记忆。
- **CLAUDE.md 和 /init：** 项目的 "Claude 的 README"。建立架构、技术栈和约定。这是最有影响力的文件。
- **包含什么：** 关于编写有效 CLAUDE.md 内容的实用指导（保持在 150 行以下，专注于 Claude 需要知道的内容）。
- **Rules：** `.claude/rules/` 中的路径范围约定。Rules 是乘数 — 它们自动应用于每个匹配的文件，无需开发者努力即可强制执行一致性。单个 `backend-testing.md` rule 确保每个测试永远遵循相同的模式。

**为什么是 Medium 级别：** 项目记忆将 Claude 从无状态工具转换为上下文感知的协作者。但仅知识本身不会创建工作流。

### Part 4: 结构化工作流（幻灯片 25–28，级别：Medium）
**目的：** 防止浪费精力并提高执行质量的系统方法。
- **任务列表：** 将复杂工作分解为可跟踪的步骤。防止范围漂移并确保完整性。
- **模型选择：** 选择正确的模型（Opus 用于架构，Sonnet 用于实现，Haiku 用于快速任务）优化成本和质量。

**为什么仍然是 Medium 级别：** 工作流很重要但相对简单的概念。它们建立在 Part 3 的项目记忆之上并更系统地使用它。升级到 High 来自领域知识。

### Part 5: 领域知识（幻灯片 29–33，级别：High）
**目的：** 可重用的、按需的专业知识。Skills 是静态记忆（CLAUDE.md/Rules）和动态 agents 之间的桥梁。
- **什么是 Skills：** Skills 作为打包的领域知识，Claude 在相关时加载。渐进式披露的概念。
- **创建 Skills：** 实践：为 TodoApp 构建 `frontend-conventions` skill，教授 Tailwind 令牌、组件模式和侧边栏集成。
- **Skill Frontmatter 和调用：** 技术细节：YAML frontmatter、手动与自动发现调用、`context: fork` 选项。

**为什么是 High 级别：** Skills 是第一个 "乘数" 概念 — 一个 skill 定义改进其领域中的每个未来交互。但 skills 是被动知识；它们需要 agents 变为主动。

### Part 6: Agentic Engineering（幻灯片 34–46，级别：High）
**目的：** 本 Presentation 涵盖的目的地。自主的、专门的 agents 协调端到端构建功能。
- **什么是 Agents：** 具有约束工具和预加载 skills 的专门 subagents 的概念。
- **Frontend Engineer Agent：** 具体的 agent，使用 TodoApp 的前端约定、将路由添加到侧边栏、遵循设计令牌。前后对比展示转换。
- **Backend Engineer Agent：** 后端的并行 agent — 遵循 FastAPI 路由模式、SQLAlchemy 模型、编写匹配现有风格的测试。
- **Commands 和编排：** 顶峰模式：Command → Agent → Skills。单个 `/add-feature` command 协调 frontend + backend agents，每个都有自己的 skills，以交付完整功能。这是建筑巅峰。
- **Hooks 和 MCP：** 生命周期自动化（预提交检查、声音通知）和外部工具集成。最后的自动化层。
- **Command → Agent → Skills：** 完整架构图。显示所有部分如何连接：commands 调用 agents，agents 加载 skills，skills 提供知识。这是 "High level" 理解幻灯片。

**为什么是 High 级别：** 本部分涵盖本 Presentation 教授的最高价值实践。之前的所有内容都是为此奠定基础。编排和 agentic 工作流代表本课程涵盖的上限 — 完整 Pro（多 agent 团队、高级编排模式）超出本 Presentation 的范围。

### High Level 幻灯片（幻灯片 44）
庆祝时刻。显示完整的 TodoApp 配置：
- CLAUDE.md 用于项目上下文
- Rules 用于路径范围约定
- Skills 用于领域知识
- Agents 用于一致执行
- Commands 用于编排工作流
- Hooks 用于生命周期自动化
- MCP servers 用于外部工具

### 附录（幻灯片 47+，无权重）
**目的：** 参考材料。每个 command、setting 和配置选项。无权重因为这些是参考查找，不是旅程里程碑。包括：工具使用、所有 slash commands、commit/PR 工作流、自定义选项、调试技巧和黄金规则。

## 编辑幻灯片时如何使用此框架

创建或修改幻灯片时，考虑：

1. **这个概念在旅程的什么位置？** 关于 "提示中更好的错误消息" 的幻灯片属于 Part 2（提示，Low 级别）。关于 "agent 记忆范围" 的幻灯片属于 Part 6（agentic，High 级别）。

2. **前后对比是什么？** 每个重要幻灯片应该隐含或明确展示对比：Low 级别（vibe coding）会发生什么与使用此技术会发生什么。使用 TodoApp 使其具体化。

3. **级别分配感觉正确吗？** 级别转换发生在 Part 章节边界。章节内的单个幻灯片继承章节的级别。

4. **它建立在前面的内容之上吗？** Skills 假设开发者已经了解 CLAUDE.md 和 Rules。Agents 假设他们了解 Skills。Commands 假设他们了解 Agents。在章节之前不要引用概念。

5. **使用 TodoApp。** 抽象解释会失去受众。显示实际的 `routes/todos.py` 代码、实际的 `Sidebar.tsx` 组件、实际的 `CLAUDE.md` 内容。运行示例使框架变得有形。

## 级别转换参考表

| 幻灯片 | 幻灯片名称 | data-level | 级别标签 |
|-------|-----------|------------|-------------|
| 10 | Better Prompting（章节分隔符） | `data-level="low"` | Low |
| 18 | Project Memory（章节分隔符） | `data-level="medium"` | Medium |
| 29 | Domain Knowledge（章节分隔符） | `data-level="high"` | High |
| 34 | Agentic Engineering（章节分隔符） | `data-level="high"` | High |

所有其他幻灯片从之前设置的最后一个 `data-level` 属性继承级别。幻灯片 1–9（介绍 + 先决条件）没有级别并保持条隐藏直到幻灯片 2 显示 "Low"（幻灯片 2–9 低于幻灯片 10 的第一个级别转换，所以条显示空/零直到幻灯片 10）。

**注意：** 主 Presentation（`presentation/index.html`）上限为 **High** 级别 — 不使用 `data-level="pro"`。Pro 刻度在旅程条上保持可见作为理论上限，但填充永远不会达到它。视频 Presentation（`1-video-workflow.html`）上限为 **Medium** 级别。

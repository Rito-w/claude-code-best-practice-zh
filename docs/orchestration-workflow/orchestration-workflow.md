<!-- 翻译标记：orchestration-workflow/orchestration-workflow.md - 已翻译 -->
# Orchestration Workflow

本文档描述 **Command → Agent (with Skill) → Skill** 编排工作流，通过天气数据获取和 SVG 渲染系统演示。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 系统概述

Weather 系统在单个编排工作流中演示了两种不同的 Skill 模式：
- **Agent Skills**（预加载）：`weather-fetcher` 在启动时作为领域知识注入到 `weather-agent`
- **Skills**（独立）：`weather-svg-creator` 由 Command 通过 Skill 工具直接调用

这展示了 **Command → Agent → Skill** 架构模式，其中：
- Command 编排工作流并处理用户交互
- Agent 使用其预加载的 Skill 获取数据
- Skill 独立创建视觉输出

## 组件摘要

| 组件 | 角色 | 示例 |
|-----------|------|---------|
| **Command** | 入口点，用户交互 | [`/weather-orchestrator`](.claude/commands/weather-orchestrator.md) |
| **Agent** | 使用预加载 Skill 获取数据（Agent Skill） | [`weather-agent`](.claude/agents/weather-agent.md) 带有 [`weather-fetcher`](.claude/skills/weather-fetcher/SKILL.md) |
| **Skill** | 独立创建输出（Skill） | [`weather-svg-creator`](.claude/skills/weather-svg-creator/SKILL.md) |

## 流程图

```
╔══════════════════════════════════════════════════════════════════╗
║              ORCHESTRATION WORKFLOW                              ║
║           Command  →  Agent  →  Skill                            ║
╚══════════════════════════════════════════════════════════════════╝

                         ┌───────────────────┐
                         │  User Interaction │
                         └─────────┬─────────┘
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  /weather-orchestrator — Command (Entry Point)      │
         └─────────────────────────┬───────────────────────────┘
                                   │
                              Step 1
                                   │
                                   ▼
                      ┌────────────────────────┐
                      │  AskUser — C° or F°?   │
                      └────────────┬───────────┘
                                   │
                         Step 2 — Agent tool
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-agent — Agent ● skill: weather-fetcher     │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          Returns: temp + unit
                                   │
                         Step 3 — Skill tool
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-svg-creator — Skill ● SVG card + output    │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          ┌────────┴────────┐
                          │                 │
                          ▼                 ▼
                   ┌────────────┐    ┌────────────┐
                   │weather.svg │    │ output.md  │
                   └────────────┘    └────────────┘
```

## 组件详情

### 1. Command

#### `/weather-orchestrator` (Command)
- **位置**: `.claude/commands/weather-orchestrator.md`
- **目的**: 入口点 — 编排工作流并处理用户交互
- **操作**:
  1. 询问用户温度单位偏好（摄氏度/华氏度）
  2. 通过 Agent 工具调用 weather-agent
  3. 通过 Skill 工具调用 weather-svg-creator
- **Model**: haiku

### 2. 带有预加载 Skill 的 Agent（Agent Skill）

#### `weather-agent` (Agent)
- **位置**: `.claude/agents/weather-agent.md`
- **目的**: 使用其预加载的 Skill 获取天气数据
- **Skills**: `weather-fetcher`（作为领域知识预加载）
- **可用工具**: WebFetch, Read
- **Model**: sonnet
- **颜色**: green
- **Memory**: project

Agent 在启动时将 `weather-fetcher` 预加载到其 Context 中。它遵循 Skill 的指令获取温度并将值返回给 Command。

### 3. Skill

#### `weather-svg-creator` (Skill)
- **位置**: `.claude/skills/weather-svg-creator/SKILL.md`
- **目的**: 创建视觉 SVG 天气卡片并写入输出文件
- **调用**: 通过 Command 的 Skill 工具调用（不预加载到任何 Agent）
- **输出**:
  - `orchestration-workflow/weather.svg` — SVG 天气卡片
  - `orchestration-workflow/output.md` — 天气摘要

### 4. 预加载 Skill

#### `weather-fetcher` (Skill)
- **位置**: `.claude/skills/weather-fetcher/SKILL.md`
- **目的**: 获取实时温度数据的指令
- **数据源**: 迪拜，阿联酋的 Open-Meteo API
- **输出**: 温度值和单位（摄氏度或华氏度）
- **注意**: 这是 Agent Skill — 预加载到 `weather-agent`，不直接调用

## 执行流程

1. **用户调用**: 用户运行 `/weather-orchestrator` Command
2. **用户提示**: Command 询问用户首选温度单位（摄氏度/华氏度）
3. **Agent 调用**: Command 通过 Agent 工具调用 `weather-agent`
4. **Skill 执行**（在 Agent Context 内）:
   - Agent 遵循 `weather-fetcher` Skill 指令从 Open-Meteo 获取温度
   - Agent 将温度值和单位返回给 Command
5. **SVG 创建**: Command 通过 Skill 工具调用 `weather-svg-creator`
   - Skill 在 `orchestration-workflow/weather.svg` 创建 SVG 天气卡片
   - Skill 将摘要写入 `orchestration-workflow/output.md`
6. **结果显示**: 向用户显示摘要，包括：
   - 请求的温度单位
   - 获取的温度
   - SVG 卡片位置
   - 输出文件位置

## 执行示例

```
Input: /weather-orchestrator
├─ Step 1: 询问：Celsius 还是 Fahrenheit?
│  └─ User: Celsius
├─ Step 2: Agent tool → weather-agent
│  ├─ Preloaded Skill:
│  │  └─ weather-fetcher (领域知识)
│  ├─ 从 Open-Meteo 获取 → 26°C
└─ Step 3: Skill tool → weather-svg-creator
   ├─ 创建：orchestration-workflow/weather.svg
   └─ 写入：orchestration-workflow/output.md
```

## 关键要点

- **分离关注点**: Command 处理编排，Agent 获取数据，Skill 创建输出
- **Skill 重用**: `weather-fetcher` 作为领域知识预加载，`weather-svg-creator` 作为独立 Skill 调用
- **清晰流程**: 每个组件都有明确定义的角色和输出
- **可扩展**: 此模式可轻松适应其他数据获取和可视化工作流

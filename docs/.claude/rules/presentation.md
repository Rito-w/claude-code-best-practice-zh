<!-- 翻译标记：.claude/rules/presentation.md - 已翻译 -->
# Glob: presentation/**

## 委托规则

任何更新、修改或修复 Presentation（`presentation/index.html`）的请求必须由 `presentation-curator` Agent 处理。始终通过 Task 工具将 Presentation 工作委托给此 Agent — 不要直接编辑 Presentation。

```
Task(subagent_type="presentation-curator", description="...", prompt="...")
```

## 为什么

Presentation-curator Agent 有三个预加载的 Skills，使其与 Presentation 的结构、样式和概念框架保持同步。它还在每次执行后自我进化，更新自己的 Skills 以防止知识漂移。绕过此 Agent 可能会破坏幻灯片编号、级别转换或样式一致性。

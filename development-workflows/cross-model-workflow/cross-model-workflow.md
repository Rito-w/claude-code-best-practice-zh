<!-- 翻译标记：development-workflows/cross-model-workflow/cross-model-workflow.md - 已翻译 -->
# Cross-Model (Claude Code + Codex) Workflow

基于 [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) 和 [codex-cli-best-practice](https://github.com/shanraisshan/codex-cli-best-practice)

## 工作流

```
┌─────────────────────────────────────────────────────────────────────────┐
│              CROSS-MODEL CLAUDE CODE + CODEX WORKFLOW                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  STEP 1: PLAN                                          Claude Code      │
│  ─────────────                                         Opus 4.6         │
│  在 Plan Mode 中打开 Claude Code（Terminal 1）。         Plan Mode        │
│  Claude 通过 AskUserQuestion 采访你。                                   │
│  生成带有测试门的分阶段计划。                                            │
│                                                                         │
│  输出：plans/{feature-name}.md                                          │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 2: QA REVIEW                                     Codex CLI        │
│  ──────────────────                                    GPT-5.4          │
│  在另一个 Terminal 中打开 Codex CLI（Terminal 2）。                       │
│  Codex 根据实际代码库审查计划。                                           │
│  插入中间阶段（"Phase 2.5"）                                             │
│  带有 "Codex Finding" 标题。                                              │
│  添加到计划 — 从不重写原始阶段。                                          │
│                                                                         │
│  输出：plans/{feature-name}.md (已更新)                                  │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 3: IMPLEMENT                                     Claude Code      │
│  ──────────────────                                    Opus 4.6         │
│  启动新的 Claude Code 会话（Terminal 1）。                                │
│  你逐阶段实现                                                          │
│  每个阶段都有测试门。                                                    │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 4: VERIFY                                        Codex CLI        │
│  ────────────────                                      GPT-5.4          │
│  启动新的 Codex CLI 会话（Terminal 2）。                                  │
│  Codex 验证实现                                                          │
│  对照计划。                                                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Cross-Model 工作流在生产环境中的实际样子

![Cross-Model Workflow](assets/cross-model-workflow.png)

*Last Updated: 2026-03-06*

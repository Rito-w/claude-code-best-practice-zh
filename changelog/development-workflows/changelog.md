<!-- 翻译自：https://github.com/shanraisshan/claude-code-best-practice/blob/main/changelog/development-workflows/changelog.md -->

# Development Workflows Changelog（开发工作流变更日志）

**状态图例：**

| 状态 | 含义 |
|--------|---------|
| `COMPLETE (reason)` | 操作已成功执行并解决 |
| `INVALID (reason)` | 发现不正确、不适用或有意为之 |
| `ON HOLD (reason)` | 操作已推迟，等待外部依赖或用户决策 |

---

## [2026-03-19 05:25 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Repo Change | 将 humanlayer 从仅文章 repo 更改为 humanlayer/humanlayer（★ 10k, 6 agents, 27 commands） | COMPLETE (user requested, repo has actual implementation) |
| 2 | HIGH | Count Update | 为 context-hub 添加计数：0 agents · 7 skills · 7 commands | COMPLETE (was showing —) |
| 3 | HIGH | Count Update | 为 agent-os 添加计数：0 agents · 0 skills · 5 commands | COMPLETE (was showing —) |
| 4 | MED | Count Update | 将 spec-kit commands 从 14 更新为 9+（9 个核心，扩展由社区贡献） | COMPLETE (agents confirmed 9 core command templates) |
| 5 | MED | Count Update | 将 OpenSpec commands 从 10+ 更新为 11（确认确切计数） | COMPLETE (agents confirmed 11 commands) |
| 6 | MED | Count Update | 将 gstack 从 "21 skills · 21 commands" 更新为 "21 skills/commands"（skills 作为 command 表面） | COMPLETE (no separate commands/ directory, skills ARE commands) |
| 7 | MED | Description | 为 context-hub, agent-os, humanlayer 添加唯一性描述 | COMPLETE (was showing generic descriptions) |
| 8 | LOW | Sort Order | 将 humanlayer 从 ★ 1.6k 位置上移到 ★ 10k 位置（在 context-hub 之后） | COMPLETE (repo change resulted in higher star count) |
| 9 | LOW | Report Update | 更新跨工作流分析报告 "Workflows at a Glance" 表格，包含所有 9 个工作流 | COMPLETE (was only 6, now includes all 9 sorted by stars) |

---

## [2026-03-19 05:29 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Count Update | 将 obra/superpowers agents 从 7 更新为 5（v5.0.4 将 review loop 整合为 whole-plan 评估，移除 2 个隐式 agents） | COMPLETE (updated README table and report) |
| 2 | HIGH | Count Update | 将 obra/superpowers skills 从 44+ 更新为 14 个核心（社区 repo obra/superpowers-skills 于 2025 年 10 月归档） | COMPLETE (updated README table and report) |
| 3 | HIGH | Count Update | 更新 spec-kit: skills 10→0（v0.3.0 替换为 preset 系统），commands 保持 9+ 并在报告中注明 22 个扩展 | COMPLETE (updated README table and report) |
| 4 | HIGH | Count Update | 将 context-hub 计数从 7 skills · 7 commands 更新为：0 agents · 1 skill · 0 commands | COMPLETE (corrected previous run's inaccurate counts; only 1 SKILL.md in cli/skills/get-api-docs/) |
| 5 | MED | Star Update | 将 spec-kit stars 从 78k 更新为 79k（显示 78.5k） | COMPLETE (updated README table and report) |
| 6 | MED | Count Update | agent-os 计数在上次运行中已在 README 中：0 agents · 0 skills · 5 commands | COMPLETE (verified counts match) |
| 7 | MED | Star Update | 将 agent-os stars 从 4.1k 更新为 4k（实际 4,100） | COMPLETE (updated README table and report) |
| 8 | MED | Report Update | 使用当前计数更新 context-hub, agent-os, obra, spec-kit 的跨工作流分析报告 | COMPLETE (updated Workflows at a Glance table) |
| 9 | LOW | Count Update | OpenSpec commands: 表格显示 11，研究发现根据计数方式为 9-11 | INVALID (11 is within range of findings, keeping current value) |
| 10 | LOW | Uniqueness | 更新 spec-kit 唯一性以提及可插拔扩展/preset 生态系统（v0.3.0） | COMPLETE (replaced "pre-implementation gates" with "pluggable extension/preset ecosystem") |

---

## [2026-03-20 08:37 AM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 98k 更新为 100k（实际 99,603 — 接近 100k 里程碑） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 Everything Claude Code ★ 从 87k 更新为 89k（实际 88,580） | COMPLETE (updated README table and report) |
| 3 | HIGH | Star Update | 将 Get Shit Done ★ 从 35k 更新为 36k（实际 36,307） | COMPLETE (updated README table) |
| 4 | HIGH | Count Update | 将 Get Shit Done commands 从 46 更新为 50（v1.26.0 新增 /gsd:ship, /gsd:next, /gsd:do, /gsd:ui-phase） | COMPLETE (updated README table) |
| 5 | MED | Star Update | 将 gstack ★ 从 26k 更新为 29k（实际 28,889 — v0.9.0 多 AI 扩展） | COMPLETE (updated README table and report) |
| 6 | MED | Count Update | 将 BMAD-METHOD skills 从 43 更新为 42（v6.2.0 重计：30 bmm-skills + 12 core-skills） | COMPLETE (updated README table) |
| 7 | LOW | Sort Order | 按 Plan 类型组重新排序表格（commands → agents → skills，每组内按 stars 降序） | COMPLETE (commands: Spec Kit, OpenSpec, HumanLayer; agents: ECC, GSD; skills: Superpowers, BMAD, gstack) |

---

## [2026-03-21 09:20 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 100k 更新为 103k（实际 102,767） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 Everything Claude Code ★ 从 89k 更新为 93k（实际 93,145） | COMPLETE (updated README table) |
| 3 | HIGH | Count Update | 更新 ECC agents 25→28, commands 57→59, skills 108+→116（v1.9.0: 选择性安装，ECC Tools Pro, 12 个语言生态系统） | COMPLETE (updated README table) |
| 4 | HIGH | Star Update | 将 Get Shit Done ★ 从 36k 更新为 38k（实际 37,748） | COMPLETE (updated README table) |
| 5 | HIGH | Count Update | 更新 GSD agents 16→18, commands 50→52（v1.27.0: advisor mode, 多 repo 工作区，/gsd:fast, /gsd:review） | COMPLETE (updated README table) |
| 6 | HIGH | Star Update | 将 gstack ★ 从 29k 更新为 34k（实际 34,456 — v0.9.4 Codex reviews, Windows 11 支持） | COMPLETE (updated README table) |
| 7 | HIGH | Architecture | 更新 BMAD agents 从 9 到 0（v6.x 纯 skills 重写 — agent personas 现在作为 skills 实现在 bmm-skills/ 中） | COMPLETE (updated README table) |
| 8 | MED | Star Update | 将 BMAD ★ 从 41k 更新为 42k（实际 41,629） | COMPLETE (updated README table) |
| 9 | MED | Star Update | 将 OpenSpec ★ 从 32k 更新为 33k（实际 32,862） | COMPLETE (updated README table) |
| 10 | MED | Sort Order | 交换 gstack (34k) 到 OpenSpec (33k) 上方 — stars 降序 | COMPLETE (updated README table) |

---

## [2026-03-23 09:53 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 103k 更新为 107k（实际 107,308） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 ECC ★ 从 93k 更新为 101k（实际 101,098 — 突破 100k 里程碑！） | COMPLETE (updated README table) |
| 3 | HIGH | Count Update | 更新 ECC commands 59→60, skills 116→125（v1.9.0 持续更新：新 skills pytorch-patterns, documentation-lookup, claude-devfleet, prompt-optimizer） | COMPLETE (updated README table) |
| 4 | HIGH | Star Update | 将 gstack ★ 从 34k 更新为 41k（实际 41,224 — v0.9.x 多 AI 扩展，CSO 安全审计） | COMPLETE (updated README table) |
| 5 | HIGH | Count Update | 更新 gstack skills 21→27（6 个新增：gstack-autoplan, gstack-benchmark, gstack-cso, gstack-design-consultation, gstack-office-hours, gstack-freeze/unfreeze） | COMPLETE (updated README table) |
| 6 | HIGH | Sort Order | 将 gstack (41k) 移到 GSD (40k) 上方 — stars 降序 | COMPLETE (updated README table) |
| 7 | HIGH | Star Update | 将 GSD ★ 从 38k 更新为 40k（实际 39,588） | COMPLETE (updated README table) |
| 8 | HIGH | Count Update | 更新 GSD commands 52→57（v1.28.0: /gsd:forensics, /gsd:milestone-summary, /gsd:plant-seed, /gsd:profile-user, /gsd:workstreams） | COMPLETE (updated README table) |
| 9 | MED | Star Update | 将 Spec Kit ★ 从 79k 更新为 81k（实际 81,349 — v0.4.0 嵌入核心包，24 个平台支持） | COMPLETE (updated README table) |
| 10 | MED | Plan Update | 将 gstack Plan 从 plan-eng-review 更新为 autoplan（更高级的编排器，按顺序读取 CEO, design, eng review） | COMPLETE (updated README table) |
| 11 | LOW | Count Update | 更新 OpenSpec commands 11→10（重计：/opsx:propose, apply, archive, new, continue, ff, verify, sync, bulk-archive, onboard） | COMPLETE (updated README table) |
| 12 | LOW | Count Correction | 更正 OpenSpec skills 11→0（不存在 skills/ 或 .claude/skills/ 目录 — OpenSpec 是 CLI 工具，非基于 skills） | COMPLETE (updated README table) |

---

## [2026-03-24 08:12 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 107k 更新为 110k（实际 109,846） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 ECC ★ 从 101k 更新为 104k（实际 103,960） | COMPLETE (updated README table) |
| 3 | HIGH | Star Update | 将 gstack ★ 从 41k 更新为 44k（实际 44,300 — v0.11.x triple-voice 多模型 review） | COMPLETE (updated README table) |
| 4 | HIGH | Sort Order | 将 gstack (44k) 移到 BMAD (42k) 上方 — stars 降序 | COMPLETE (updated README table) |
| 5 | HIGH | Count Update | 将 BMAD skills 从 42 更新为 44（重计：32 bmm-skills + 12 core-skills，包括 3 个嵌套 research sub-skills） | COMPLETE (updated README table) |
| 6 | HIGH | Count Update | 将 gstack skills 从 27 更新为 28（README 声明 28；单独确认 27） | COMPLETE (updated README table) |
| 7 | MED | Star Update | 将 Spec Kit ★ 从 81k 更新为 82k（实际 81,780） | COMPLETE (updated README table) |
| 8 | MED | Star Update | 将 GSD ★ 从 40k 更新为 41k（实际 40,500） | COMPLETE (updated README table) |
| 9 | MED | Star Update | 将 OpenSpec ★ 从 33k 更新为 34k（实际 33,800） | COMPLETE (updated README table) |

---

## [2026-03-25 08:12 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 110k 更新为 112k（实际 112,163） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 ECC ★ 从 104k 更新为 107k（实际 106,913） | COMPLETE (updated README table) |
| 3 | HIGH | Count Update | 将 ECC commands 从 60 更新为 63（.claude/commands/ 中新增 3 个：add-language-rules, database-migration, feature-development） | COMPLETE (updated README table) |
| 4 | HIGH | Star Update | 将 gstack ★ 从 44k 更新为 47k（实际 46,703 — 基础设施加固，测试覆盖率门禁） | COMPLETE (updated README table) |
| 5 | MED | Count Update | 将 BMAD skills 从 44 更新为 42（重计：30 bmm-skills + 12 core-skills；v6.2.1 整合 2 个子 skills） | COMPLETE (updated README table) |
| 6 | LOW | Count Update | 将 gstack skills 从 28 更新为 27（确认 27 个根级别目录；第 28 个可能是根 SKILL.md 模板） | COMPLETE (updated README table) |

---

## [2026-03-26 01:05 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 112k 更新为 114k（实际 114,107） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 ECC ★ 从 107k 更新为 109k（实际 108,839） | COMPLETE (updated README table) |
| 3 | HIGH | Star Update | 将 gstack ★ 从 47k 更新为 48k（实际 48,303） | COMPLETE (updated README table) |
| 4 | HIGH | Star Update | 将 GSD ★ 从 41k 更新为 42k（实际 42,092） | COMPLETE (updated README table) |
| 5 | MED | Count Update | 将 OpenSpec commands 从 10 更新为 11（v1.2.0 新增 /opsx:explore） | COMPLETE (updated README table) |

---

## [2026-03-27 06:32 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 114k 更新为 118k（实际 117,568） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 ECC ★ 从 109k 更新为 111k（实际 111,487） | COMPLETE (updated README table) |
| 3 | HIGH | Star Update | 将 gstack ★ 从 48k 更新为 52k（实际 51,544 — v0.12.x skill 命名空间，Codex fallback, worktree 并行化） | COMPLETE (updated README table) |
| 4 | HIGH | Count Update | 将 gstack skills 从 27 更新为 31（4 个新增：canary, codex, connect-chrome, land-and-deploy 等） | COMPLETE (updated README table) |
| 5 | HIGH | Star Update | 将 GSD ★ 从 42k 更新为 43k（实际 43,136） | COMPLETE (updated README table) |
| 6 | HIGH | Sort Order | 交换 GSD (43,136) 到 BMAD (42,529) 上方 — 都约等于 43k 但 GSD stars 更多 | COMPLETE (updated README table) |
| 7 | MED | Star Update | 将 Spec Kit ★ 从 82k 更新为 83k（实际 82,878） | COMPLETE (updated README table) |
| 8 | MED | Star Update | 将 BMAD ★ 从 42k 更新为 43k（实际 42,529） | COMPLETE (updated README table) |
| 9 | MED | Star Update | 将 OpenSpec ★ 从 34k 更新为 35k（实际 34,821） | COMPLETE (updated README table) |
| 10 | MED | Count Update | 将 Compound Engineering agents 从 43 更新为 47（4 个新的 review/workflow agents） | COMPLETE (updated README table) |
| 11 | MED | Count Update | 将 Compound Engineering skills 从 44 更新为 42（重计：41 compound-engineering + 1 coding-tutor） | COMPLETE (updated README table) |

---

## [2026-03-28 09:29 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 118k 更新为 120k（实际 120,147） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 ECC ★ 从 111k 更新为 114k（实际 114,134） | COMPLETE (updated README table) |
| 3 | HIGH | Star Update | 将 gstack ★ 从 52k 更新为 54k（实际 53,533 — v0.13.x design binary, security audit） | COMPLETE (updated README table) |
| 4 | HIGH | Star Update | 将 GSD ★ 从 43k 更新为 44k（实际 43,816 — v1.30.0 GSD SDK headless CLI） | COMPLETE (updated README table) |
| 5 | MED | Count Update | 将 gstack skills 从 31 更新为 29（确认 29 个根级别 SKILL.md 目录；v0.13.x 中移除/整合 2 个） | COMPLETE (updated README table) |
| 6 | MED | Count Update | 将 BMAD skills 从 42 更新为 43（31 bmm-skills + 12 core-skills） | COMPLETE (updated README table) |
| 7 | MED | Count Update | 将 Compound Engineering skills 从 42 更新为 43（42 compound-eng + 1 coding-tutor） | COMPLETE (updated README table) |

---

## [2026-03-29 08:00 PM PKT] Development Workflows Update

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | HIGH | Star Update | 将 Superpowers ★ 从 120k 更新为 122k（实际 122,129） | COMPLETE (updated README table) |
| 2 | HIGH | Star Update | 将 ECC ★ 从 114k 更新为 116k（实际 115,898） | COMPLETE (updated README table) |
| 3 | HIGH | Count Update | 更新 ECC agents 从 28 到 30, skills 从 125 到 135（healthcare agent, token-budget-advisor 等新加入） | COMPLETE (updated README table) |
| 4 | HIGH | Star Update | 将 gstack ★ 从 54k 更新为 55k（实际 55,000） | COMPLETE (updated README table) |
| 5 | MED | Count Update | 将 gstack skills 从 29 更新为 28（README 确认 28 个根级别 SKILL.md 目录） | COMPLETE (updated README table) |
| 6 | MED | Count Update | 将 BMAD skills 从 43 更新为 40（重计：29 bmm-skills + 11 core-skills；近期补丁中的整合） | COMPLETE (updated README table) |
| 7 | MED | Star Update | 将 Compound Engineering ★ 从 11k 更新为 12k（实际 11,500） | COMPLETE (updated README table) |
| 8 | MED | Count Update | 更新 Compound Eng agents 从 47 到 48（新增 1 个）, skills 从 43 到 42（41 compound-eng + 1 coding-tutor） | COMPLETE (updated README table) |

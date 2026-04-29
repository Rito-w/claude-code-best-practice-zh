<!-- 
 翻译来源：https://github.com/shanraisshan/claude-code-best-practice/blob/main/changelog/skill-collections/changelog.md
 翻译时间：2026-04-30 02:00 CST
 翻译版本：v1.0
-->
# Skill Collections 变更日志

**状态图例：**

| 状态 | 含义 |
|--------|---------|
| `COMPLETE (原因)` | 操作已执行并成功解决 |
| `INVALID (原因)` | 发现不正确、不适用或有意为之 |
| `ON HOLD (原因)` | 操作延期，等待外部依赖或用户决策 |

---

## [2026-04-28 04:39 PM PKT] Skill Collections 更新

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | LOW | 首次运行 | 在 README 中创建 SKILL COLLECTIONS 部分，包含 5 个仓库：anthropics/skills (125k/17)、wshobson/agents (35k/152)、mattpocock/skills (33k/17)、K-Dense-AI/scientific-agent-skills (20k/134)、VoltAgent/awesome-agent-skills (19k/1,100+ curated) | COMPLETE (首次播种来自 research-agent 的调研结果，2026-04-28 会话) |

---

## [2026-04-29 12:52 AM PKT] Skill Collections 更新

| # | 优先级 | 类型 | 操作 | 状态 |
|---|----------|------|--------|--------|
| 1 | MEDIUM | Star | 更新 mattpocock/skills ★ 从 33k 到 36k（36,476 精确值） | NEW |
| 2 | MEDIUM | Count | 更新 mattpocock/skills skill 数量从 17 到 18（新增 setup-matt-pocock-skills，deprecated/ 文件夹于 2026-04-28 重组） | NEW |
| 3 | LOW | Star | 更新 wshobson/agents ★ 从 35k 到 34k（34,477 精确值 — 略有下降） | NEW |
| 4 | MEDIUM | Sort | 将 mattpocock/skills 行移到 wshobson/agents 行上方（因 star 变化导致排名交换） | NEW |
| 5 | LOW | Count | 更新 VoltAgent/awesome-agent-skills 精选数量从 1,100+ 到 930+（实际 README  bullet 解析；badge 高估约 170） | NEW |
| 6 | LOW | No Change | anthropics/skills (125k/17) 和 K-Dense-AI/scientific-agent-skills (20k/134) — 数值匹配，无需编辑 | COMPLETE (已验证，无漂移) |

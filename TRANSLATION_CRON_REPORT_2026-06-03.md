# 翻译同步报告 — 2026-06-03

**源仓库版本**: v2.1.160 → v2.1.161 (Jun 03, 2026)
**上次同步基准**: v2.1.159 (Jun 02, 2026, commit 9a60dd4)
**本次提交**: be42fbf

## 本次翻译变更

### claude-settings.md (核心翻译)
- **acceptEdits** (v2.1.160): 新增安全守卫 — 写入构建工具配置文件（`.npmrc`、`.yarnrc*`、`bunfig.toml`、`.bazelrc`、`.pre-commit-config.yaml`、`.devcontainer/` 等）和 shell 启动文件（`.zshenv`、`.zlogin`、`.bash_login`）以及 `~/.config/git/` 之前始终会提示
- **CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE**: 标记为 v2.1.160 已移除（no-op）
- 徽章: v2.1.159 → v2.1.160

### claude-commands.md
- 徽章: v2.1.160 → v2.1.161（无实质内容变更）

### claude-skills.md
- 徽章: v2.1.160 → v2.1.161（无实质内容变更）

### claude-subagents.md
- 徽章: v2.1.160 → v2.1.161（无实质内容变更）

### README.md
- Ultrareview Location: `/ultrareview` → `/code-review ultra`（官方文档更新，`/ultrareview` 保留为别名）
- 徽章: v2.1.160 → v2.1.161
- 工作流表格星数刷新（Superpowers 216k→206k, ECC 204k→192k, 等）
- 技能集合星数刷新
- Agent Collections 星数刷新

### Changelog 文件
- 所有 9 个 changelog 文件从上游同步（v2.1.160 和 v2.1.161 漂移检查条目）
- 自动生成的英文报告，无需翻译

## 翻译状态
- **翻译覆盖率**: 100% (核心文档)
- **上游版本**: v2.1.161 ✅ 已追平
- **仓库**: https://github.com/Rito-w/claude-code-best-practice-zh

## 备注
- v2.1.161 主要是徽章刷新和无漂移检查，无新翻译内容
- `workflowKeywordTriggerEnabled` 和 `ultracode` 设置项在上游存在但翻译文件中缺失，留待后续补充翻译

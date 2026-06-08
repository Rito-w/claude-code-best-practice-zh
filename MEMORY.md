
## 翻译报告 2026-06-09

**上游版本**: v2.1.168 (Jun 08, 2026)  
**翻译时间**: 2026-06-09 02:00-02:36 PKT

### 翻译文件

| 文件 | 上游行数 | 译文行数 | 变更类型 |
|------|----------|----------|----------|
| claude-subagents.md | 56 | 56 | badge 日期更新 |
| claude-skills.md | 53 | 53 | badge + 2行描述翻译 |
| claude-cli-startup-flags.md | 231 | 231 | 5行描述翻译 |
| claude-commands.md | 136 | 136 | 全文翻译（大量新内容） |
| claude-settings.md | 1238 | 1240 | 全文翻译（v2.1.168 新设置项） |

### 新增内容（v2.1.168）
- `settings.md`: 新增 `gcpAuthRefresh`, `skillOverrides` 对象格式, `disableRemoteControl`, `disableAgentView`, `disableWorkflows`, `workflowKeywordTriggerEnabled`, `ultracode` 等设置
- `commands.md`: 新增 `/ultraplan`, `/autofix-pr`, `/install-slack-app` 等命令
- `sandbox`: `autoAllowBashIfSandboxed` v2.1.139 改进说明
- `permissions`: 新增 Deny rule glob patterns (v2.1.166)
- `MCP`: 新增 `.mcp.json` hot-reload, per-server timeout floor, `allowAllClaudeAiMcps`

### 提交: 564ce76

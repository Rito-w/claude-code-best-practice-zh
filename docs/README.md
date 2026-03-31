# 📖 Claude Code 最佳实践 - 中文翻译版

📖 **本项目是 [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) 的中文翻译版本。**

## 🚀 快速开始

### 什么是 Claude Code？

Claude Code 是 Anthropic 推出的命令行 AI 编程助手，可以帮助你：

- 自动完成代码编写
- 代码审查和优化
- 调试和问题解决
- 文档生成
- 测试编写

### 安装

```bash
# macOS
brew install claude-code

# Windows
winget install Anthropic.ClaudeCode

# Linux
npm install -g @anthropic-ai/claude-code
```

### 配置

安装完成后运行：

```bash
claude
```

按照提示完成配置。

## 📚 文档导航

### 最佳实践
- [**Commands**](best-practice/claude-commands.md) - Commands 最佳实践指南
- [**Skills**](best-practice/claude-skills.md) - Skills 配置和使用
- [**SubAgents**](best-practice/claude-subagents.md) - 子代理最佳实践
- [**Settings**](best-practice/claude-settings.md) - 设置完整参考
- [**MCP**](best-practice/claude-mcp.md) - MCP 协议和服务
- [**Memory**](best-practice/claude-memory.md) - 记忆功能指南
- [**CLI 参数**](best-practice/claude-cli-startup-flags.md) - 启动参数参考

### 实现指南
- [**Agent Teams**](implementation/claude-agent-teams-implementation.md) - Agent Teams 实现
- [**Commands**](implementation/claude-commands-implementation.md) - Commands 实现
- [**Skills**](implementation/claude-skills-implementation.md) - Skills 实现
- [**SubAgents**](implementation/claude-subagents-implementation.md) - SubAgents 实现
- [**定时任务**](implementation/claude-scheduled-tasks-implementation.md) - 定时任务实现

### 工作流
- [**编排工作流**](orchestration-workflow/orchestration-workflow.md) - 工作流编排
- [**RPI 工作流**](development-workflows/rpi/rpi-workflow.md) - RPI 开发流程
- [**跨模型工作流**](development-workflows/cross-model-workflow/cross-model-workflow.md) - 跨模型协作

### 技术报告
- [**使用情况和限制**](reports/claude-usage-and-rate-limits.md) - 使用限制详解
- [**Agent Memory**](reports/claude-agent-memory.md) - Agent 记忆机制
- [**MCP 对比**](reports/claude-in-chrome-v-chrome-devtools-mcp.md) - MCP 对比分析

### 使用技巧
- [**Boris 技巧**](tips/claude-boris-10-tips-01-feb-26.md) - 10 个实用技巧
- [**Thariq 技巧**](tips/claude-thariq-tips-17-mar-26.md) - 高级技巧

### 安装教程
- [**macOS**](tutorial/day0/mac.md) - macOS 安装指南
- [**Windows**](tutorial/day0/windows.md) - Windows 安装指南
- [**Linux**](tutorial/day0/linux.md) - Linux 安装指南

## 🌐 语言切换

每个文档都有中英文两个版本：
- `xxx.md` - **中文翻译版** 🇨🇳
- `xxx.en.md` - **英文原版** 🇺🇸

## 📊 翻译统计

| 项目 | 数量 |
|------|------|
| 中文翻译 | 103 个 |
| 英文原版 | 104 个 |
| 总计 | 207 个 |

## 📝 翻译规则

### 保留英文（不翻译）
- 技术术语：Agent, Command, Skill, Hook, MCP, LSP, Context
- 模式名称：Plan Mode, Act Mode, Vibe Coding
- 工具名称：Claude Code, GitHub, Git, npm, Node.js
- 所有代码块和行内代码

### 翻译成中文
- 文档正文
- 标题和描述
- 列表项
- 说明文字

## ⏰ 自动更新

- **定时任务**: 每天凌晨 2:00 (Asia/Shanghai)
- **自动检查**: 检查源仓库变更
- **增量翻译**: 仅翻译变更的文件

## 🛠️ 相关项目

- **源仓库**: https://github.com/shanraisshan/claude-code-best-practice
- **翻译仓库**: https://github.com/Rito-w/claude-code-best-practice-zh
- **翻译技能**: ~/.openclaw/workspace/skills/claude-code-translator/

## 📞 反馈

发现翻译问题？欢迎提交 Issue 或 PR！

---
**翻译版本**: 1.0.0  
**最后更新**: 2026-03-31  
**源仓库版本**: 查看 [原仓库](https://github.com/shanraisshan/claude-code-best-practice)

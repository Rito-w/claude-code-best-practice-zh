# Claude Code 最佳实践

> 本仓库包含使用 Claude Code 的全面指南和最佳实践

## 📖 关于本仓库

本仓库是学习使用 Claude Code（Anthropic 的命令行 AI 编程助手）的综合资源。它包含：

- **最佳实践** - 有效使用 Claude Code 的指南
- **实现指南** - 如何实施特定功能
- **工作流** - 实际项目开发流程
- **技术报告** - 技术细节和限制分析
- **使用技巧** - 提升效率的小技巧

## 🚀 快速开始

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

安装后运行：

```bash
claude
```

按照提示完成配置。

## 📚 文档导航

### 核心文档

#### 最佳实践
- [**Commands**](best-practice/claude-commands.md) - 命令使用最佳实践
- [**Skills**](best-practice/claude-skills.md) - Skills 配置指南
- [**SubAgents**](best-practice/claude-subagents.md) - 子代理使用指南
- [**Settings**](best-practice/claude-settings.md) - 设置完整参考
- [**MCP**](best-practice/claude-mcp.md) - MCP 协议和服务
- [**Memory**](best-practice/claude-memory.md) - 记忆功能指南
- [**CLI 启动参数**](best-practice/claude-cli-startup-flags.md)

#### 实现指南
- [**Agent Teams**](implementation/claude-agent-teams-implementation.md)
- [**Commands**](implementation/claude-commands-implementation.md)
- [**Skills**](implementation/claude-skills-implementation.md)
- [**SubAgents**](implementation/claude-subagents-implementation.md)
- [**定时任务**](implementation/claude-scheduled-tasks-implementation.md)

#### 工作流
- [**编排工作流**](orchestration-workflow/orchestration-workflow.md)
- [**RPI 工作流**](development-workflows/rpi/rpi-workflow.md)
- [**跨模型工作流**](development-workflows/cross-model-workflow/cross-model-workflow.md)

### 技术报告
- [**使用情况和限制**](reports/claude-usage-and-rate-limits.md)
- [**Agent Memory**](reports/claude-agent-memory.md)
- [**MCP 对比**](reports/claude-in-chrome-v-chrome-devtools-mcp.md)
- [**SDK vs CLI**](reports/claude-agent-sdk-vs-cli-system-prompts.md)
- [**全局 vs 项目设置**](reports/claude-global-vs-project-settings.md)

### 使用技巧
- **Boris 技巧系列**（8 篇）
- **Thariq 技巧**

### 视频摘要
- **Lenny's Podcast**
- **The Pragmatic Engineer**
- **Y Combinator**
- **Ryan Peterman 访谈**

## 🌐 语言版本

本仓库提供中英文两个版本：
- `.md` - 中文翻译版
- `.en.md` - 英文原版

## 📊 统计

- 中文翻译：103 个文件
- 英文原版：104 个文件
- 总计：207 个文件

## 🔗 相关链接

- [Claude Code 官网](https://claude.ai/code)
- [Anthropic 文档](https://docs.anthropic.com/claude-code)
- [源仓库](https://github.com/shanraisshan/claude-code-best-practice)

## 🤝 贡献

欢迎提交 Issue 和 PR！

## 📄 许可证

MIT License

---
**最后更新**: 2026-03-31

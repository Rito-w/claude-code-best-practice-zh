<!-- TRANSLATED: Auto-translated from source -->

# MCP Servers Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../.mcp.json)

MCP (Model Context Protocol) servers 通过连接外部 tools、databases 和 APIs 来扩展 Claude Code。本指南涵盖日常使用的推荐 servers 和配置最佳实践。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 日常使用的 MCP Servers

> *"一开始装了 15 个 MCP servers 以为越多越好，结果每天只用 4 个。"* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/) (682 upvotes)

| MCP Server | 作用 | Resources |
|------------|-------------|-----------|
| [**Context7**](https://github.com/upstash/context7) | 获取最新的 library docs 到 context 中。防止因过时的训练数据导致 APIs hallucinated | [Reddit: "by far the best MCP for coding"](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | Browser automation — 自主实现、测试和验证 UI features。Screenshots, navigation, form testing | [Reddit: essential for frontend](https://reddit.com/r/mcp/comments/1m59pk0/) · [Docs](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | 将 Claude 连接到你的真实 Chrome browser — 检查 console, network, DOM。Debug what users actually see | [Reddit: "game changer" for debugging](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [Comparison Report](../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | 为任何 GitHub repo 获取 structured wiki-style documentation — architecture, API surface, relationships | [Reddit: "put it behind a gateway with Context7"](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | 从 prompts 生成 architecture diagrams, flowcharts, 和 system designs 作为手绘 Excalidraw sketches | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

Research (Context7/DeepWiki) -> Debug (Playwright/Chrome) -> Document (Excalidraw)

---

## 配置

MCP servers 在项目 root 的 `.mcp.json`（project-scoped）或 `~/.claude.json`（user-scoped）中配置。

### Server Types

| Type | Transport | Example |
|------|-----------|---------|
| **stdio** | Spawns a local process | `npx`, `python`, binary |
| **http** | Connects to a remote URL | HTTP/SSE endpoint |

### `.mcp.json` 示例

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

对环境变量使用 expansion 来处理 secrets，而不是将 API keys 提交到 `.mcp.json`：

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

### MCP Servers 的 Settings

`.claude/settings.json` 中的这些 settings 控制 MCP server approval：

| Key | Type | Description |
|-----|------|-------------|
| `enableAllProjectMcpServers` | boolean | Auto-approve 所有 `.mcp.json` servers 而无需 prompting |
| `enabledMcpjsonServers` | array | Allowlist of specific server names to auto-approve |
| `disabledMcpjsonServers` | array | Blocklist of specific server names to reject |

### MCP Tools 的 Permission Rules

MCP tools 在 permission rules 中遵循 `mcp__<server>__<tool>` naming convention：

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

---

## MCP Scopes

MCP servers 可以在三个级别定义：

| Scope | Location | Purpose |
|-------|----------|---------|
| **Project** | `.mcp.json` (repo root) | Team-shared servers, committed to git |
| **User** | `~/.claude.json` (`mcpServers` key) | Personal servers across all projects |
| **Subagent** | Agent frontmatter (`mcpServers` field) | Servers scoped to a specific subagent |

优先级：Subagent > Project > User

---

## Sources

- [MCP Servers — Claude Code Docs](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [5 MCPs that have genuinely made me 10x faster — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [MCP Server Overload Discussion — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)

<!-- TRANSLATED: Auto-translated from source -->

# Claude Memory

通过 CLAUDE.md files 实现 Persistent context — 如何编写它们以及它们在 monorepos 中如何加载。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1. 编写一个好的 CLAUDE.md

一个结构良好的 CLAUDE.md 是改善 Claude Code 对项目输出的最有效方法。Humanlayer 有一篇优秀的指南，涵盖了应该包含什么、如何结构化它以及常见陷阱。

- [Humanlayer - Writing a good Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## 2. 大型 Monorepos 中的 CLAUDE.md

在 monorepo 中使用 Claude Code 时，了解 CLAUDE.md files 如何加载到 context 中对于有效组织项目指令至关重要。

<p align="center">
  <a href="https://x.com/bcherny/status/2016339448863355206"><img src="assets/claude-memory/claude-memory-monorepo.jpg" alt="CLAUDE.md loading in monorepos" width="600"></a>
</p>

### 两种加载机制

Claude Code 使用两种不同的机制来加载 CLAUDE.md files：

#### Ancestor Loading（向上遍历 directory tree）

当你启动 Claude Code 时，它从当前 working directory 向 filesystem root **向上**遍历，并加载沿途找到的每个 CLAUDE.md。这些文件在**启动时立即**加载。

#### Descendant Loading（向下遍历 directory tree）

当前 working directory 下方子目录中的 CLAUDE.md files **在启动时不加载**。它们仅在 session 期间 Claude 读取这些子目录中的文件时才被包含。这称为 **lazy loading**。

### 示例 Monorepo Structure

考虑一个典型的 monorepo，为不同组件设有单独的目录：

```
/mymonorepo/
├── CLAUDE.md          # Root-level instructions（所有组件共享）
├── frontend/
│   └── CLAUDE.md      # Frontend-specific instructions
├── backend/
│   └── CLAUDE.md      # Backend-specific instructions
└── api/
    └── CLAUDE.md      # API-specific instructions
```

### Scenario 1: 从 Root Directory 运行 Claude Code

当你从 `/mymonorepo/` 运行 Claude Code 时：

```bash
cd /mymonorepo
claude
```

| File | Loaded at Launch? | Reason |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | Yes | 它是你的 current working directory |
| `/mymonorepo/frontend/CLAUDE.md` | No | 仅在读取/编辑 `frontend/` 中的文件时加载 |
| `/mymonorepo/backend/CLAUDE.md` | No | 仅在读取/编辑 `backend/` 中的文件时加载 |
| `/mymonorepo/api/CLAUDE.md` | No | 仅在读取/编辑 `api/` 中的文件时加载 |

### Scenario 2: 从 Component Directory 运行 Claude Code

当你从 `/mymonorepo/frontend/` 运行 Claude Code 时：

```bash
cd /mymonorepo/frontend
claude
```

| File | Loaded at Launch? | Reason |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | Yes | 它是 ancestor directory |
| `/mymonorepo/frontend/CLAUDE.md` | Yes | 它是你的 current working directory |
| `/mymonorepo/backend/CLAUDE.md` | No | directory tree 的不同分支 |
| `/mymonorepo/api/CLAUDE.md` | No | directory tree 的不同分支 |

### 关键要点

1. **Ancestors always load at startup** — Claude 向上遍历 directory tree 并加载找到的所有 CLAUDE.md files。这确保你始终可以访问 root-level, repository-wide instructions。

2. **Descendants load lazily** — Subdirectory CLAUDE.md files 仅在你与这些子目录中的文件交互时加载。这防止无关 context 使 session 膨胀。

3. **Siblings never load** — 如果你在 `frontend/` 中工作，你不会得到 `backend/CLAUDE.md` 或 `api/CLAUDE.md` 加载到 context 中。

4. **Global CLAUDE.md** — 你也可以在 home folder 的 `~/.claude/CLAUDE.md` 放置 CLAUDE.md，它适用于所有 Claude Code sessions 而不管 project。

### 为什么这种设计适用于 Monorepos

- **Shared instructions propagate down** — Root-level CLAUDE.md 包含 repository-wide conventions、coding standards 和 common patterns，适用于所有地方。

- **Component-specific instructions stay isolated** — Frontend 开发者不需要 backend-specific instructions  cluttering their context，反之亦然。

- **Context is optimized** — 通过 lazily loading descendant CLAUDE.md files，Claude Code 避免在 startup 时加载 potentially hundreds of kilobytes of irrelevant instructions。

### 最佳实践

1. **Put shared conventions in root CLAUDE.md** — Coding standards, commit message formats, PR templates, 和其他 repository-wide guidelines。

2. **Put component-specific instructions in component CLAUDE.md** — Framework-specific patterns, component architecture, testing conventions unique to that component。

3. **Use CLAUDE.local.md for personal preferences** — 将它添加到 `.gitignore` 用于不应与团队共享的 instructions。

---

## Sources

- [Claude Code Documentation - How Claude Looks Up Memories](https://code.claude.com/docs/en/memory#how-claude-looks-up-memories)
- [Boris Cherny on X - Clarification on CLAUDE.md Loading](https://x.com/bcherny/status/2016339448863355206)
- [Humanlayer - Writing a good Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

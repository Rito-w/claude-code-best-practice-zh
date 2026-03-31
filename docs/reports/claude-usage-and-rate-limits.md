<!-- TRANSLATED: Auto-translated from source -->

# Claude Code: Usage, Rate Limits & Extra Usage

了解 Claude Code 中的 usage limits 如何工作以及命中限制时如何继续工作。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Overview

subscription plans（Pro, Max 5x, Max 20x）上的 Claude Code 有 usage limits，在 rolling window 上重置。三个 built-in slash commands 帮助监控和管理 usage：

| Command | Description | Available To |
|---------|-------------|--------------|
| `/usage` | 检查 plan limits 和 rate limit status | Pro, Max 5x, Max 20x |
| `/extra-usage` | 配置 pay-as-you-go overflow 当命中 limits 时 | Pro, Max 5x, Max 20x |
| `/cost` | 显示当前 session 的 token usage 和 spending | API key users |

---

## `/usage` — Check Your Limits

显示当前 plan 的 usage limits 和 rate limit status。用于检查在命中 limit 之前还有多少 capacity。

---

## `/extra-usage` — Keep Working Past Limits

`/extra-usage` command 配置 **pay-as-you-go overflow billing**，这样当你命中 plan 的 rate limits 时 Claude Code 继续无缝工作，而非阻止你。

### How It Works

1. 你命中 plan 的 rate limit（limits 每 5 小时重置）
2. 如果 extra usage 已启用且有可用 funds，Claude Code 继续无中断
3. Overflow tokens 按 **standard API rates** 计费，与 subscription fee 分开

### Setting It Up

CLI 中的 `/extra-usage` command 会引导你完成配置。你也可以在 claude.ai 的 **Settings > Usage** 上配置：

1. Enable extra usage
2. Add a payment method
3. 设置 **monthly spending cap**（或选择 unlimited）
4. 可选添加 **prepaid funds** 并设置 balance 低于 threshold 时 auto-reload

### Key Details

| Detail | Value |
|--------|-------|
| Daily redemption limit | $2,000/day |
| Billing | 与 subscription 分开，按 standard API rates |
| Limit reset window | 每 5 小时 |

### Known Issue

截至 2026 年 2 月，`/extra-usage` CLI command 是 [undocumented](https://github.com/anthropics/claude-code/issues/12396) 且可能打开 sign-in window 而没有清晰的 configuration options。目前通过 **claude.ai web interface** 配置是更可靠的路径。

---

## `/cost` — Session Spending (API Users)

对于使用 API key 认证的用户（不是 subscription plan），`/cost` 显示：

- 当前 session 的 total cost
- API duration 和 wall time
- Token usage breakdown
- Code changes made

此 command 与 Pro/Max subscription users 无关。

---

## Fast Mode and Extra Usage

Fast mode（`/fast`）使用 Claude Opus 4.6 和更快的 output。它与 extra usage 有特殊的 billing relationship：

- Fast mode usage **始终从第一个 token 开始计费到 extra usage**
- 即使你的 subscription plan 上还有 remaining usage 也适用
- Fast mode 不消耗 plan 的 included rate limits

这意味着你需要 extra usage enabled and funded 才能使用 `/fast`。

---

## CLI Startup Flags

两个 startup flags 与 usage budgets 相关（仅 API key users，print mode）：

| Flag | Description |
|------|-------------|
| `--max-budget-usd <AMOUNT>` | API calls 停止前的 Maximum dollar amount |
| `--max-turns <NUMBER>` | Limit number of agentic turns |

参见 [CLI Startup Flags Reference](claude-cli-startup-flags.md) 获取完整列表。

---

## Best Practices

1. **Monitor usage regularly** — 使用 `/usage` 检查 remaining capacity
2. **Set up extra usage** — 避免 work interruptions 当 hitting limits 时
3. **Set spending caps** — 防止 unexpected charges
4. **Use fast mode wisely** — 记住它始终 billed to extra usage

---

## Sources

- [Claude Code Usage Documentation](https://code.claude.com/docs/en/usage)
- [Claude Code Rate Limits](https://code.claude.com/docs/en/rate-limits)
- [GitHub Issue: /extra-usage undocumented](https://github.com/anthropics/claude-code/issues/12396)

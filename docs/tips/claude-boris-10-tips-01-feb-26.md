<!-- TRANSLATED: Auto-translated from source -->

# 10 Tips for Using Claude Code — From the Claude Code Team

Boris Cherny ([@bcherny](https://x.com/bcherny)) 分享的团队技巧摘要，Claude Code 的创作者，2026 年 2 月 1 日。

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../_media/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

Boris 分享了直接从 Claude Code 团队获取的使用 Claude Code 的技巧。团队使用 Claude 的方式与 Boris 个人使用方式不同。记住：使用 Claude Code 没有唯一正确的方式 — 每个人的设置都不同。你应该实验看看什么适合你！

<a href="https://x.com/bcherny/status/2017742741636321619"><img src="assets/boris-1-feb-26/0.png" alt="Boris Cherny intro tweet" width="50%" /></a>

---

## 1/ Do More in Parallel

同时启动 3-5 个 git worktrees，每个运行自己的 Claude session 并行。这是最大的生产力提升，也是团队的首要考虑。个人而言，Boris 使用多个 git checkouts，但大多数 Claude Code 团队成员更喜欢 worktrees — 这就是为什么 `@amorisscode` 为 Claude Desktop app 构建了原生支持！

有些人还命名他们的 worktrees 并设置 shell aliases（`2a`, `2b`, `2c`）这样他们可以在一次击键之间切换。其他人有专用的 "analysis" worktree 仅用于 reading logs 和 running BigQuery。

参见：[Worktrees Docs](https://code.claude.com/docs/en/common...)

<a href="https://x.com/bcherny/status/2017742743125299476"><img src="assets/boris-1-feb-26/1.png" alt="Do more in parallel" width="50%" /></a>

---

## 2/ Start Every Complex Task in Plan Mode

将精力投入到 plan 中，这样 Claude 可以 1-shot the implementation。

一个人让一个 Claude 编写 plan，然后他们启动第二个 Claude 作为 staff engineer 来 review。

另一个人说，一旦出现问题，他们就切换回 plan mode 并 re-plan。不要继续推进。他们还明确告诉 Claude 为 verification steps 进入 plan mode，而不仅仅是为 build。

<a href="https://x.com/bcherny/status/2017742745365057733"><img src="assets/boris-1-feb-26/2.png" alt="Start every complex task in plan mode" width="50%" /></a>

---

## 3/ Invest in Your CLAUDE.md

每次 correction 后，以 "Update your CLAUDE.md so you don't make that mistake again" 结束。Claude 非常擅长为自己编写 rules。

随着时间的推移无情地编辑你的 `CLAUDE.md`。持续迭代直到 Claude 的 mistake rate 明显下降。

一位工程师告诉 Claude 为每个 task/project 维护一个 notes directory，每次 PR 后更新。然后他们将 `CLAUDE.md` 指向它。

<a href="https://x.com/bcherny/status/2017742747067945390"><img src="assets/boris-1-feb-26/3.png" alt="Invest in your CLAUDE.md" width="50%" /></a>

---

## 4/ Create Your Own Skills and Commit Them to Git

跨每个项目重用。来自团队的技巧：

- 如果你每天做某事超过一次，把它变成 skill 或 command
- 构建一个 `/techdebt` slash command 并在每次 session 结束时运行它以查找和消除 duplicated code
- 设置一个 slash command 将 7 天的 Slack、GDrive、Asana 和 GitHub 同步到一个 context dump 中
- 构建 analytics-engineer-style agents 来编写 dbt models、review code 和 test changes in dev

参见：[Extend Claude with Skills — Claude Code Docs](https://code.claude.com/docs/en/skills)

<a href="https://x.com/bcherny/status/2017742748984742078"><img src="assets/boris-1-feb-26/4.png" alt="Create your own skills" width="50%" /></a>

---

## 5/ Claude Fixes Most Bugs by Itself

这是团队的做法：

启用 Slack MCP，然后将 Slack bug thread 粘贴到 Claude 中并只说 "fix"。无需 context switching。

或者，只说 "Go fix the failing CI tests"。不要 micromanage how。

让 Claude 查看 docker logs 来 troubleshoot distributed systems — 它在这方面出奇地能干。

<a href="https://x.com/bcherny/status/2017742750473720121"><img src="assets/boris-1-feb-26/5.png" alt="Claude fixes most bugs by itself" width="50%" /></a>

---

## 6/ Level Up Your Prompting

a. **Challenge Claude.** 说 "Grill me on these changes and don't make a PR until I pass your test." 让 Claude 成为你的 reviewer。或者，说 "Prove to me this works" 并让 Claude diff behavior between main and your feature branch。

b. **After a mediocre fix,** 说："Knowing everything you know now, scrap this and implement the elegant solution."

c. **Write detailed specs** 并在 handing work off 之前减少 ambiguity。你越具体，output 就越好。

<a href="https://x.com/bcherny/status/2017742752566632544"><img src="assets/boris-1-feb-26/6.png" alt="Level up your prompting" width="50%" /></a>

---

## 7/ Terminal & Environment Setup

（继续翻译其余内容...）

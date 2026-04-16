<!-- 
 翻译来源：https://github.com/shanraisshan/claude-code-best-practice/blob/main/tips/claude-thariq-tips-16-apr-26.md
 翻译时间：2026-04-17 02:00 CST
 翻译版本：v2.1.107
-->

# 使用 Claude Code：会话管理与 1M 上下文 — Thariq

Thariq ([@trq212](https://x.com/trq212)) 于 2026 年 4 月 16 日分享的关于管理 Claude Code 中的会话、上下文窗口和压缩的指南。

<table width="100%">
<tr>
<td><a href="../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 背景

借助 1M token 的上下文窗口，Claude Code 可以更可靠地处理更长的任务——但如果你不有意识地管理会话，也会带来上下文污染的风险。会话管理比以往任何时候都更重要：何时开始新会话、何时压缩、何时回退、以及何时委托给 Subagent。

<img src="assets/thariq-16-apr-26/1.png" alt="Thariq intro tweet" width="50%" />

<img src="assets/thariq-16-apr-26/2.png" alt="Session management intro" width="50%" />

---

## 上下文、压缩与上下文退化的快速入门

上下文窗口是模型在生成下一个回复时所能"看到"的一切。它包括你的系统提示词、到目前为止的对话、每个工具调用及其输出、以及每个已读取的文件。Claude Code 的上下文窗口为 **一百万 token**。

不幸的是，使用上下文有一个轻微的代价——**上下文退化（context rot）**。随着上下文的增长，模型性能会下降，因为注意力被分散到更多 token 上，旧的、不相关的内容开始干扰当前任务。对于 1M 上下文模型，某种程度的上下文退化大约发生在 **~300-400k token** 左右，但这高度依赖于任务——不是一条硬性规则。

上下文窗口是一个硬性限制。当你接近末尾时，你需要总结任务并在新的上下文窗口中继续——这就是**压缩（compaction）**。你也可以自己触发压缩。

<img src="assets/thariq-16-apr-26/3.png" alt="Context window diagram" width="50%" />

<img src="assets/thariq-16-apr-26/4.png" alt="Context rot explanation" width="50%" />

---

## 每个回合都是一个分支点

在 Claude 完成一个回合后，你有很多令人惊讶的选项可以决定下一步做什么：

- **继续（Continue）**——在同一会话中发送另一条消息
- **/rewind（esc esc）**——跳回到之前的某条消息，从那里重新开始
- **/clear**——开始一个新会话，通常带着你从刚才学到的东西中提炼出的简要说明
- **压缩（Compact）**——总结到目前为止的会话，然后在总结的基础上继续
- **Subagent**——将下一块工作委托给拥有自己干净上下文的 agent，只拉回它的结果

虽然最自然的做法是继续，但其他四个选项的存在是为了帮助你管理上下文。

<img src="assets/thariq-16-apr-26/5.png" alt="Compaction and branching diagram" width="50%" />

<img src="assets/thariq-16-apr-26/6.png" alt="Five options after a turn" width="50%" />

每个选项携带的现有上下文的量不同：

| 新会话 | 压缩 | Subagent | 回退 | 继续 |
|:---:|:---:|:---:|:---:|:---:|
| 仅你的简要说明 | 有损总结 | 全部 + 结果 | 保留前缀，截断尾部 | 保留一切 |
| *完全不携带* | | | | *全部携带* |

<img src="assets/thariq-16-apr-26/7.png" alt="Context carry-forward spectrum" width="50%" />

---

## 何时开始新会话

新的 1M 上下文窗口意味着你现在可以更可靠地完成更长的任务——例如，从头构建一个全栈应用。但仅仅因为你的模型还没有耗尽上下文，并不意味着你不应该开始新会话。

**一般经验法则：当你开始一个新任务时，你也应该开始一个新会话。**

一个灰色地带是，当你想做相关的任务时，其中一些上下文仍然有必要，但不是全部。例如，为你刚刚实现的功能编写文档。虽然你可以开始一个新会话，但 Claude 必须重新读取文件，这会更慢且更昂贵。由于文档编写可能不是一个高度依赖智力的任务，额外的上下文带来的效率提升可能是值得的。

<img src="assets/thariq-16-apr-26/8.png" alt="When to start a new session" width="50%" />

---

## 回退而不是纠正

如果 Thariq 必须选择一个能标志良好上下文管理的习惯，那就是**回退**。

在 Claude Code 中，双击 Esc（或运行 `/rewind`）可以让你跳回到任何之前的消息，并从那里重新提示。该点之后的消息将从上下文中删除。

**纠正**（在失败的尝试 A 之后说"不，试试 B"）会让失败的尝试留在上下文中：
> context = 读取 + 2 次失败尝试 + 2 次纠正 + 修复

**回退**（回到失败尝试之前，用你学到的东西重新提示）更干净：
> context = 读取 + 一个知情的提示 + 修复

回退通常是更好的方法。例如，Claude 读取了五个文件，尝试了一种方法，但没有成功。你的本能可能是输入"那不行，试试 X。"但更好的做法是回退到文件读取之后，用你学到的东西重新提示："不要用方法 A，foo 模块没有暴露那个接口——直接用 B。"

你也可以使用**"从这里总结"**让 Claude 总结它的学习成果并创建一条交接消息，有点像来自未来的 Claude（尝试过但失败了）给过去的自己的消息。

<img src="assets/thariq-16-apr-26/9.png" alt="Correcting vs rewinding diagram" width="50%" />

<img src="assets/thariq-16-apr-26/10.png" alt="Rewind with summarize from here" width="50%" />

---

## 压缩 vs 新会话

当会话变长时，你有两种方式来减轻负担：`/compact` 或 `/clear`（重新开始）。它们感觉相似，但行为完全不同。

**压缩**要求模型总结到目前为止的对话，然后用该总结替换历史记录。这是有损的——你信任 Claude 来决定什么重要，但你不必自己写任何东西。Claude 可能会更彻底地包含重要的学习成果或文件。你也可以通过传递指令来引导它（`/compact 专注于 auth 重构，删除测试调试`）。

- **任务中途**，保持势头——细节可以模糊
- 低成本，继续进行

**新会话 + 简要说明**（`/clear`）意味着*你*写下重要的东西（"我们正在重构 auth 中间件，约束是 X，重要的文件是 A 和 B，我们已经排除了方法 Y"），然后干净地开始。这需要更多工作，但产生的上下文是*你*决定相关的。

- **高风险**的下一步——在 100K 的探索中找到了一个事实
- 更多工作，更精确

<img src="assets/thariq-16-apr-26/11.png" alt="Compacting vs fresh sessions" width="50%" />

<img src="assets/thariq-16-apr-26/12.png" alt="Compact vs fresh diagram" width="50%" />

---

## 是什么导致了糟糕的压缩？

如果你运行很多长时间运行的会话，你可能已经注意到有时压缩效果特别差。糟糕的压缩可能发生在模型无法预测你的工作方向时。

例如，在一次长时间的调试会话后自动压缩触发了，并总结了调查结果。你的下一条消息是"现在修复我们在 bar.ts 中看到的那个其他警告。"但因为会话专注于调试，那个其他警告可能已经从总结中被丢弃了。

这尤其困难，因为由于上下文退化，模型在压缩时正处于其最不智能的状态。有了百万上下文，你有更多时间主动 `/compact`，并描述你想做什么。

<img src="assets/thariq-16-apr-26/13.png" alt="Bad compact diagram" width="50%" />

<img src="assets/thariq-16-apr-26/14.png" alt="Bad compact explanation" width="50%" />

---

## Subagent 与干净上下文窗口

Subagent 是一种上下文管理的形式，当你提前知道一块工作会产生大量你不再需要的中间输出时非常有用。

当 Claude 通过 Agent 工具生成一个 Subagent 时，该 Subagent 会获得自己全新的上下文窗口。它可以做任意多的工作，然后综合其结果，只将最终报告返回给父级。

心理测试：**我还需要这个工具输出吗，还是只需要结论？**

探索噪音在 Subagent 退出时被垃圾回收——20 次文件读取、12 次 grep、3 条死胡同——只有最终报告返回到父级上下文。

虽然 Claude Code 会自动调用 Subagent，但你可能想明确告诉它这样做。例如：

- "启动一个 Subagent 来根据以下 spec 文件验证这项工作的结果"
- "启动一个 Subagent 来阅读另一个代码库，总结它如何实现 auth 流程，然后用同样的方式实现"
- "启动一个 Subagent 来根据我的 git 变更编写这个功能的文档"

<img src="assets/thariq-16-apr-26/15.png" alt="Subagent context diagram" width="50%" />

<img src="assets/thariq-16-apr-26/16.png" alt="Subagent explanation" width="50%" />

<img src="assets/thariq-16-apr-26/17.png" alt="When to use subagents" width="50%" />

---

## 总结

当 Claude 结束一个回合而你即将发送新消息时，你有一个决策点。随着时间的推移，Claude 会自己处理这个问题，但现在这是你可以引导 Claude 输出的方式之一。

| 情况 | 使用 | 原因 |
|-----------|-----------|-----|
| 相同任务，上下文仍然相关 | **继续** | 窗口中的一切仍然有负载作用——不要花钱重建它 |
| Claude 走了一条错误的路 | **回退**（双击 Esc） | 保留有用的文件读取，丢弃失败的尝试，用你学到的东西重新提示 |
| 任务中途，但会话充满了过时的调试/探索 | **/compact \<提示\>** | 低投入；Claude 决定什么重要。如果需要，用提示引导它 |
| 开始一个真正的新任务 | **/clear** | 零退化；你完全控制携带什么 |
| 下一步将生成大量输出，而你只需要结论 | **Subagent** | 中间工具噪音留在子级的上下文中；只有结果返回 |

<img src="assets/thariq-16-apr-26/18.png" alt="Summary" width="50%" />

<img src="assets/thariq-16-apr-26/19.png" alt="Decision table" width="50%" />

---

## 来源

- [Thariq (@trq212) on X — 2026 年 4 月 16 日](https://x.com/trq212)

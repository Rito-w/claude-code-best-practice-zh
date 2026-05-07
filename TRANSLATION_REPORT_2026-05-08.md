# 翻译日报 — 2026-05-08 (周五)

## 上游更新 (a56ea4a → 4671f03)

### 变更文件

| 文件 | 类型 | 操作 |
|------|------|------|
| `README.md` | 徽章/数据 | 版本号更新，star 数更新 (93k→95k, 197→185 agents) |
| `changelog/agent-collections/changelog.md` | 数据 | 新增 2 条 agent collections 更新日志 |
| `presentation/assets/llm/*.svg, *.jpg` | 资源 | 3 个新 LLM 插图 (llm-basic.svg, llm-advanced.svg, tokens.jpg) |
| `presentation/claude-code-best-practice/index.html` | 演示文稿 | **新增 3 张幻灯片 + 重编号** |

### 翻译详情

**新增 3 张 LLM 基础幻灯片（插入到原 slide 11 位置，后续 slide +3 重编号）：**

| Slide | 英文标题 | 中文标题 |
|-------|---------|---------|
| 11 | One token at a time | 一次一个 token |
| 12 | Tokens, not words | Token，不是词语 |
| 13 | Tokens in, tokens out | Token 进，Token 出 |

内容涵盖：自回归生成原理、tokenization 概念、输入输出共享词表。

## 翻译仓库状态

- **上游 HEAD**: `4671f03` chore(agent-collections): scheduled refresh
- **翻译 HEAD**: `1cae304` translate: 3 new LLM slides (upstream 4671f03)
- **状态**: ✅ 已同步并推送

## 备注

- 上游出现 500 错误（临时），通过 upstream remote fetch 成功获取
- README 和 changelog 的变更主要是数字/徽章更新，无实质文本翻译需求

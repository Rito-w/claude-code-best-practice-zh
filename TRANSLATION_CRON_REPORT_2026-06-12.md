# Claude Code 翻译日报 — 2026-06-12

## 同步版本
- 上游版本: **v2.1.175** (Jun 12, 2026)
- 上次同步: v2.1.173 (Jun 11, 2026)
- 同步时间: 2026-06-13 02:00 AM (Asia/Shanghai)

## 变更文件 (5 个)
1. **README.md** — 版本徽章 v2.1.175，Star 计数更新，agent-skills 位置调整
2. **best-practice/claude-commands.md** — 命令描述更新（5 项）
3. **best-practice/claude-settings.md** — 新增设置项（4 类）
4. **best-practice/claude-skills.md** — 版本徽章更新
5. **best-practice/claude-subagents.md** — 版本徽章更新

## 关键翻译变更
### claude-commands.md
- `/color [color|default]` — 新增"不带参数运行可随机选择颜色"
- `/context` → `/context [all]` — 新增"传入 `all` 展开完整详情"
- `/remote-env` — 描述更新为"选择云端代理的默认环境"
- `/cd [path]` → `/cd <path>` — 参数语法更新
- `/clear` → `/clear [name]` — 新增"传入可选的 `name` 为之前的对话命名"

### claude-settings.md
- 新增 `advisorModel` 设置 — 服务器端 advisor 工具使用的模型
- 新增 `Cd` 权限规则 — 控制 `/cd` 命令可导航的目录
- 新增 `fable` 模型选项 — Claude Fable 5 长时推理模型
- 新增 Fable 环境变量 — `ANTHROPIC_DEFAULT_FABLE_MODEL` 等 4 个

## 状态
✅ 翻译完成
✅ 已提交
✅ 已推送到 GitHub

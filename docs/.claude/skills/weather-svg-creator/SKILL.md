---
name: weather-svg-creator
description: 创建显示迪拜当前温度的 SVG 天气卡片。将 SVG 写入 orchestration-workflow/weather.svg 并更新 orchestration-workflow/output.md。
---

<!-- 翻译标记：.claude/skills/weather-svg-creator/SKILL.md - 已翻译 -->

# Weather SVG Creator Skill

为迪拜，阿联酋创建可视化 SVG 天气卡片并写入输出文件。

## 任务

你将从调用上下文接收温度值和单位（摄氏度或华氏度）。创建 SVG 天气卡片并写入 SVG 和 markdown 摘要。

## 指令

1. **创建 SVG** — 使用 [reference.md](reference.md) 中的 SVG 模板，用实际值替换占位符
2. **写入 SVG 文件** — 读取然后写入到 `orchestration-workflow/weather.svg`
3. **写入摘要** — 读取然后写入到 `orchestration-workflow/output.md`，使用 [reference.md](reference.md) 中的 markdown 模板

## 规则

- 使用提供的确切温度值和单位 — 不要重新获取或修改
- SVG 必须是自包含且有效的
- 两个输出文件都放在 `orchestration-workflow/` 目录中

## 其他资源

- 对于 SVG 模板、输出模板和设计规范，参见 [reference.md](reference.md)
- 对于示例输入/输出对，参见 [examples.md](examples.md)

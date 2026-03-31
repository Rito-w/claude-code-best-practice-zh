---
name: time-skill
description: 显示巴基斯坦标准时间（PKT, UTC+5）的当前时间。当用户询问当前时间、巴基斯坦时间或 PKT 时使用。
user-invocable: true
---

<!-- 翻译标记：.claude/skills/time-skill/SKILL.md - 已翻译 -->

# Time Skill

此 Skill 显示巴基斯坦标准时间（PKT）的当前日期和时间。

## 任务

显示巴基斯坦标准时间（UTC+5）的当前日期和时间。

## 指令

1. **获取当前时间**：运行以下 bash 命令：
   ```
   TZ='Asia/Karachi' date '+%Y-%m-%d %H:%M:%S %Z'
   ```

2. **显示结果**：按以下格式显示时间：
   ```
   Current Time in Pakistan (PKT): YYYY-MM-DD HH:MM:SS PKT
   ```

## 要求

- 始终使用 `Asia/Karachi` 时区（UTC+5）
- 使用 24 小时制
- 日期和时间一起显示
- 保持输出简洁 — 无需额外说明

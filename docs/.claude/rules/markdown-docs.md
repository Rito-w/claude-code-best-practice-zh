<!-- 翻译标记：.claude/rules/markdown-docs.md - 已翻译 -->
# Glob: **/*.md

## 文档标准

- 保持文件聚焦且简洁 — 每个文件一个主题
- 在文档之间使用相对链接（例如 `../best-practice/claude-memory.md`），而不是绝对 GitHub URL
- 在最佳实践和报告文档顶部包含返回导航链接（参见现有文件模式）
- 添加新概念或报告时，更新 README.md 中对应的表格（CONCEPTS 或 REPORTS）

## 结构约定

- 最佳实践文档放在 `best-practice/`
- 实现文档放在 `implementation/`
- 报告放在 `reports/`
- 提示放在 `tips/`
- Changelog 跟踪放在 `changelog/<category>/`

## 格式

- 使用表格进行结构化比较（参见 README CONCEPTS 表格作为参考）
- 链接最佳实践或实现文档时使用 `_media/tags/` 中的徽章图像以保持视觉一致性
- 保持标题层次结构 — 不要跳过级别（例如不要从 `##` 跳到 `####`）

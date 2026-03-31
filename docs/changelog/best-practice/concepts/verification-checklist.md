
# Verification Checklist — README CONCEPTS Section（验证清单 — README CONCEPTS 部分）

CONCEPTS 表格准确性验证规则。每个工作流运行期间都会检查每条规则。

## 规则

### 1. External URL Liveness（外部 URL 活性）
- **类别**: URL Accuracy
- **检查内容**: CONCEPTS 表格中的每个外部 URL（docs links）返回有效页面
- **深度**: 获取每个 URL 并确认它加载预期页面（不是重定向到错误页面）
- **对比源**: `https://code.claude.com/docs/llms.txt` 获取规范 URL 列表
- **添加日期**: 2026-03-02
- **来源**: Permissions URL `/iam` 被发现重定向到 Authentication 页面而非 Permissions

### 2. Anchor Fragment Validity（Anchor 片段有效性）
- **类别**: URL Accuracy
- **检查内容**: 任何带有 anchor fragment（`#section-name`）的 URL 与目标页面上的实际标题匹配
- **深度**: 获取页面并验证标题是否存在并带有预期 anchor
- **对比源**: 获取的页面内容
- **添加日期**: 2026-03-02
- **来源**: Rules anchor `#modular-rules-with-clauderules` 已过时；部分重命名为 `#organize-rules-with-clauderules`

### 3. Missing Docs Pages（缺失文档页面）
- **类别**: Missing Concepts
- **检查内容**: 官方文档索引（`llms.txt`）中代表用户功能的每个页面在 CONCEPTS 表格中都有对应行
- **深度**: 将完整 docs 索引与 CONCEPTS 表格条目进行比较
- **对比源**: `https://code.claude.com/docs/llms.txt`
- **添加日期**: 2026-03-02
- **来源**: 发现多个缺失概念（Agent Teams, Keybindings, Model Configuration 等）

### 4. Local Badge Link Validity（本地徽章链接有效性）
- **类别**: Badge Accuracy
- **检查内容**: CONCEPTS 表格中的每个徽章目标路径（`best-practice/*.md`, `implementation/*.md`, `.claude/*/`）指向存在的文件或目录
- **深度**: 使用 Read/Glob 验证文件存在性
- **对比源**: 本地文件系统
- **添加日期**: 2026-03-02
- **来源**: 初始清单创建

### 5. Description Currency（描述时效性）
- **类别**: Description Accuracy
- **检查内容**: 每个概念的描述准确反映当前官方文档描述
- **深度**: 将 README 描述与官方页面的 meta description 或第一段进行比较
- **对比源**: 官方文档页面内容
- **添加日期**: 2026-03-02
- **来源**: Memory 描述缺少 auto memory；MCP Servers 位置缺少 `.mcp.json`

---
name: presentation-structure
description: 关于 Presentation 幻灯片格式、权重系统、导航和章节结构的知识
---

<!-- 翻译标记：.claude/skills/presentation/presentation-structure/SKILL.md - 已翻译 -->

# Presentation Structure Skill

关于 `presentation/index.html` 的 Presentation 如何结构化的知识。

## 文件位置

`presentation/index.html` — 带内联 CSS 和 JS 的单文件 HTML Presentation。

## 幻灯片格式

每个幻灯片是一个 div，带有 `data-slide`（顺序号）和可选的 `data-level`（转换点的旅程级别）：

```html
<!-- 普通幻灯片 — 从前一个 data-level 幻灯片继承级别 -->
<div class="slide" data-slide="12">
    <h1>Slide Title</h1>
    <!-- content -->
</div>

<!-- 级别转换幻灯片 — 为此幻灯片和所有后续幻灯片设置新级别 -->
<div class="slide section-slide" data-slide="10" data-level="low">
    <h1>Section Name</h1>
    <p class="section-desc">Level: Low — description of this section</p>
</div>

<!-- 标题幻灯片（居中） -->
<div class="slide title-slide" data-slide="1">
    <h1>Presentation Title</h1>
    <p class="subtitle">Subtitle text</p>
</div>
```

## Journey Bar 级别系统

Presentation 使用 4 级系统而不是累积百分比：

- 级别通过关键转换幻灯片（章节分隔符）上的 `data-level` 属性设置
- `data-level` 幻灯片之后的所有幻灯片继承该级别直到下一个转换
- 旅程条填充到 25% / 50% / 75% / 100% 分别对应 Low / Medium / High / Pro
- 条在幻灯片 1（标题幻灯片）上隐藏；从幻灯片 2 开始显示条
- 第一个 `data-level` 之前的幻灯片（幻灯片 2–9）显示空条（尚未设置级别）
- `.level-badge` 在运行时通过 JS 注入到带有 `data-level` 的幻灯片的 `<h1>` 上 — 不要在 HTML 中硬编码

### 按章节的级别转换

| 章节 | 幻灯片范围 | data-level | 条高度 |
|---------|-------------|------------|------------|
| Part 0: Introduction | Slides 1-4 | (none) | hidden / empty |
| Part 1: Prerequisites | Slides 5-9 | (none) | empty |
| Part 2: Better Prompting | Slides 10-17 | `low` | 25% |
| Part 3: Project Memory | Slides 18-24 | `medium` | 50% |
| Part 4: Structured Workflows | Slides 25-28 | (inherits medium) | 50% |
| Part 5: Domain Knowledge | Slides 29-33 | `high` | 75% |
| Part 6: Agentic Engineering | Slides 34-46 | `high` | 75% |
| Appendix | Slides 47+ | (inherits high) | 75% |

## 导航系统

- `goToSlide(n)` — 用于 TOC 链接，必须匹配实际的 `data-slide` 编号
- `totalSlides` 从 DOM 自动计算（`document.querySelectorAll('[data-slide]').length`）
- 箭头键、空格和触摸滑动用于导航
- 幻灯片计数器在左下角显示 `current / total`

## 重新编号规则

添加、删除或重新排序幻灯片后：
1. 从 1 开始按顺序重新编号所有 `data-slide` 属性
2. 更新 TOC/Journey Map 幻灯片中的所有 `goToSlide()` 调用
3. JS `totalSlides` 自动计算 — 无需手动更新
4. 验证不存在间隙或重复

## 章节分隔符格式

章节分隔符使用 `section-slide` 类。级别转换章节分隔符带有 `data-level` 并在描述中显示级别名称：

```html
<div class="slide section-slide" data-slide="10" data-level="low">
    <p class="section-number">Part 2</p>
    <h1>Better Prompting</h1>
    <p class="section-desc">Level: Low — effective prompting for real results.</p>
</div>
```

当级别转换时，JS 会在运行时将 `.level-badge`（例如 "→ Low"）注入到 `<h1>` 中 — 不要在 HTML 中手动添加这些。

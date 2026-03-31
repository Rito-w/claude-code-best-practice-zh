---
name: presentation-styling
description: 关于 Presentation 中的 CSS 类、组件模式和语法高亮的知识
---

<!-- 翻译标记：.claude/skills/presentation/presentation-styling/SKILL.md - 已翻译 -->

# Presentation Styling Skill

`presentation/index.html` 中使用的 CSS 类和 HTML 模式。

## CSS 组件类

### 布局

- `.two-col` — 带 24px 间隙的 2 列网格布局
- `.info-grid` — 用于信息卡的 2 列网格
- `.col-card` — 列内的卡片（添加 `.good` 用于绿色边框，`.bad` 用于红色边框）
- `.info-card` — 信息网格中的卡片

### 内容块

- `.trigger-box` — 带深色左边框的灰色框（用于关键概念、先决条件）
- `.how-to-trigger` — 带绿色边框的绿色框（用于 "Try This" 操作）
- `.warning-box` — 带警告边框的橙色框（用于重要警告）
- `.code-block` — 带等宽字体的深色代码显示块

### 列表

- `.use-cases` — 带图标 + 文本列表项的容器
- `.use-case-item` — 带图标和文本的单个项
- `.feature-list` — 简单的边框列表

### 标签和徽章

- `.matcher-tag` — 灰色内联药丸标签
- `.weight-badge` — 绿色药丸徽章（由 JS 自动注入到加权幻灯片）

## 代码块语法高亮

在 `.code-block` 内，使用这些 spans 进行语法着色：

```html
<div class="code-block">
<span class="comment"># This is a comment</span>
<span class="key">field_name</span>: <span class="string">value</span>
<span class="cmd">&gt;</span> command to run
</div>
```

- `.comment` — 绿色 (#6a9955) 用于注释
- `.key` — 蓝色 (#9cdcfe) 用于属性名/键
- `.string` — 橙色 (#ce9178) 用于字符串值
- `.cmd` — 黄色 (#dcdcaa) 用于命令/提示

## 幻灯片类型模式

### 带两列的内容幻灯片（好与坏）
```html
<div class="slide" data-slide="N" data-weight="5">
    <h1>Title</h1>
    <div class="two-col">
        <div class="col-card bad">
            <h4>Before (Vibe Coding)</h4>
            <!-- bad example -->
        </div>
        <div class="col-card good">
            <h4>After (Agentic)</h4>
            <!-- good example -->
        </div>
    </div>
</div>
```

不要在幻灯片 HTML 中硬编码 `<span class="weight-badge">`。Presentation JavaScript 自动注入和移除权重徽章。

### 带代码示例的内容幻灯片
```html
<div class="slide" data-slide="N">
    <h1>Title</h1>
    <div class="trigger-box">
        <h4>Key Concept</h4>
        <p>Description</p>
    </div>
    <div class="code-block"><span class="comment"># Example</span>
<span class="key">field</span>: <span class="string">value</span></div>
</div>
```

### 图标列表模式
```html
<div class="use-cases">
    <div class="use-case-item">
        <span class="use-case-icon">EMOJI</span>
        <div class="use-case-text">
            <strong>Title</strong>
            <span>Description text</span>
        </div>
    </div>
</div>
```

## Journey Bar 特定

- `.journey-bar` — 进度条下方的固定条
- `.journey-bar.hidden` — 在标题幻灯片上隐藏
- Journey bar 颜色通过 HSL 插值从红色（0%）过渡到绿色（100%）
- 权重徽章由 JS 自动注入到加权幻灯片的 `h1` 元素中

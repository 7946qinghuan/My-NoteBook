---
title: Obsidian_CSS美化方案
date: 2026-08-08
tags:
  - Obsidian
---

# Obsidian 全套常用 CSS 片段与文件分配方案

> **存放目录**：`库根目录/.obsidian/snippets/`
> 
> **管理原则**：同类组件合并，独立功能单独新建 CSS。请按照标注的文件名在 `snippets` 文件夹下新建对应的 `.css` 文件并启用。

## 一、表格全套 (`table-all-style.css`)

> 包含整张表格全宽撑满、单元格水平垂直居中、网格边框、适宜内边距，以及暗色/浅色模式下的双色隔行变色。

```css
/* ========================================================
   Obsidian 表格样式：两视图完美全宽 + 单元格全居中 + 智能双色隔行变色
   ======================================================== */

/* 1. 阅读视图表格撑满正文区域 */
.markdown-reading-view table {
  width: 100% !important;
  max-width: 100% !important;
  table-layout: auto !important;
  margin: 0 auto !important;
}

/* 2. 编辑视图/实时预览：修正外层所有相关容器，确保不塌陷、完美全宽 */
.markdown-source-view .cm-embed-block:has(.cm-table-widget),
.markdown-source-view .cm-table-widget,
.markdown-source-view .cm-table-widget-container {
  width: 100% !important;
  max-width: 100% !important;
  display: flex !important;
  justify-content: center !important;
}

/* 3. 编辑视图内部实际表格撑满 */
.markdown-source-view table {
  width: 100% !important;
  max-width: 100% !important;
  table-layout: auto !important;
  margin: 0 auto !important;
}

/* 4. 单元格居中、内边距、行高 */
table th, table td {
  text-align: center !important;
  vertical-align: middle !important;
  padding: 8px 12px;
  line-height: 1.6;
}

/* 5. 边框 */
table {
  border-collapse: collapse;
  border: 1px solid #555;
}
th, td {
  border: 1px solid #555;
}

/* 6. 【暗色模式】隔行变色 */
.theme-dark tbody tr:nth-child(even),
tbody tr:nth-child(even) {
  background-color: rgba(255, 255, 255, 0.06) !important;
}

/* 7. 【浅色模式】隔行底色 */
.theme-light tbody tr:nth-child(even) {
  background-color: rgba(0, 0, 0, 0.04) !important;
}
```

## 二、图片美化 (`image-card.css`)

> 为渲染的图片添加适度圆角与悬浮阴影，提升视觉卡片质感（同时兼容编辑与阅读模式）。

```css
/* 实时预览与阅读视图图片圆角与阴影 */
.markdown-rendered img,
.markdown-source-view .cm-image-widget img {
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.25);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

/* 悬浮微放大交互效果 */
.markdown-rendered img:hover,
.markdown-source-view .cm-image-widget img:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.35);
}
```

## 三、高亮荧光笔效果 (`text-highlight.css`)

> 增强默认 `==高亮==` 效果，并提供自定义多色高亮类（如 `<mark class="hl-green">内容</mark>`）。

```css
/* 默认高亮样式优化 */
mark {
  background-color: #ffdd44 !important;
  color: #111 !important;
  padding: 1px 5px;
  border-radius: 3px;
  font-weight: 500;
}

/* 多颜色高亮扩展类 */
mark.hl-yellow { background-color: #ffdd44 !important; color: #111 !important; }
mark.hl-green  { background-color: #89d989 !important; color: #111 !important; }
mark.hl-blue   { background-color: #82cafc !important; color: #111 !important; }
mark.hl-red    { background-color: #ffb3b3 !important; color: #111 !important; }
mark.hl-purple { background-color: #d8b4fe !important; color: #111 !important; }
```

## 四、标题美化 (`heading-style.css`)

> 为 H1 与 H2 标题增加底边分隔线，提升大纲分段的清晰度。

```css
/* 全局标题字重设定 */
.markdown-rendered h1, .markdown-rendered h2, 
.markdown-rendered h3, .markdown-rendered h4,
.cm-header-1, .cm-header-2, .cm-header-3 {
  font-weight: 600;
}

/* 一级标题下划线 */
.markdown-rendered h1,
.HyperMD-header-1 {
  border-bottom: 2px solid var(--interactive-accent, #4299e1);
  padding-bottom: 4px;
  margin-bottom: 12px;
}

/* 二级标题下划线 */
.markdown-rendered h2,
.HyperMD-header-2 {
  border-bottom: 1px solid var(--text-muted, #687280);
  padding-bottom: 3px;
  margin-bottom: 10px;
}
```

## 五、列表层级引导线 (`list-indent-line.css`)

> 在多层列表结构中添加淡色引导竖线，适合书写大纲与技术笔记。

```css
/* 嵌套列表引导竖线 */
.markdown-rendered ul ul, 
.markdown-rendered ol ul,
.cm-embed-block ul ul {
  position: relative;
}

.markdown-rendered ul ul::before,
.markdown-rendered ol ul::before {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: -12px;
  border-left: 1px dashed var(--text-faint, rgba(120, 140, 180, 0.3));
}
```

## 六、代码块美化 (`code-block.css`)

> 优化行内代码的高亮边框与整块代码框的圆角及边距。

```css
/* 行内代码样式 */
:not(pre) > code {
  padding: 2px 6px !important;
  border-radius: 4px !important;
  background-color: var(--code-background, rgba(110, 110, 110, 0.2)) !important;
  color: var(--text-accent, #e5c07b) !important;
  font-size: 0.9em;
}

/* 块级代码容器 */
.markdown-rendered pre,
.cm-s-obsidian pre.HyperMD-codeblock {
  border-radius: 8px !important;
  padding: 14px 18px !important;
  border: 1px solid var(--background-modifier-border, rgba(255, 255, 255, 0.1));
}
```

## 七、简约悬浮 UI (`ui-auto-fade.css`)

> 鼠标移开时自动降低顶部动作按钮与底部状态栏的透明度，打造无干扰写作环境。

```css
/* 顶部动作按钮淡出 */
.view-header:not(:hover) .view-actions {
  opacity: 0.15;
  transition: opacity 0.3s ease;
}

/* 底部状态栏淡化 */
.status-bar:not(:hover) .status-bar-item {
  opacity: 0.2;
  transition: opacity 0.3s ease;
}
```

## 八、美化折叠箭头 (`ui-fold-arrow.css`)

> 优化标题与列表的折叠指示图标，使其在常态下平滑淡化，鼠标悬停时高亮。

```css
.collapse-indicator,
.cm-fold-indicator {
  opacity: 0.35;
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.collapse-indicator:hover,
.cm-fold-indicator:hover {
  opacity: 1;
  color: var(--interactive-accent, #4299e1);
}
```

## 九、Callout 提示框美化 (`callout-beautify.css`)

> 增加 Callout 标注块的圆角、内边距与柔和阴影。

```css
.callout {
  border-radius: 8px !important;
  padding: 12px 16px !important;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  border: 1px solid var(--background-modifier-border);
}
```

## 十、任务复选框美化 (`checkbox-style.css`)

> 优化 Markdown `- [ ]` 复选框的尺寸、圆角与勾选后的圆滑动画。

```css
/* 任务复选框样式修正 */
.contains-task-list-item input[type="checkbox"],
.cm-task-list-item-checkbox {
  appearance: none;
  -webkit-appearance: none;
  width: 16px;
  height: 16px;
  border: 1.5px solid var(--text-muted, #888);
  border-radius: 4px;
  cursor: pointer;
  position: relative;
  vertical-align: middle;
  transition: background-color 0.15s ease, border-color 0.15s ease;
}

/* 选中状态 */
.contains-task-list-item input[type="checkbox"]:checked,
.cm-task-list-item-checkbox:checked {
  background-color: var(--interactive-accent, #4299e1);
  border-color: var(--interactive-accent, #4299e1);
}

/* 选中后的打勾对钩 */
.contains-task-list-item input[type="checkbox"]:checked::after {
  content: "";
  position: absolute;
  left: 4px;
  top: 1px;
  width: 5px;
  height: 9px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

/* 完成任务的文字文本样式（删除线加淡化） */
.task-list-item-checkbox:checked + span,
.cm-task-list-item-checkbox:checked + span {
  text-decoration: line-through;
  color: var(--text-faint, #777);
}
```

## 十一、分割线美化 (`hr-style.css`)

> 替换默认硬朗的 `<hr>` 为优雅的渐变渐隐分隔线。

```css
.markdown-rendered hr,
.cm-line:has(.cm-hr) {
  border: none !important;
  height: 2px !important;
  background: linear-gradient(to right, transparent, var(--text-faint, #666), transparent) !important;
  margin: 2em 0 !important;
  opacity: 0.6;
}
```

## 十二、标签美化 (`tag-style.css`)

> 为内置的 `#标签` 添加胶囊圆角质感与悬浮高亮效果。

```css
a.tag,
.cm-hashtag {
  background-color: var(--tag-background, rgba(66, 153, 225, 0.15)) !important;
  color: var(--tag-color, #4299e1) !important;
  border: 1px solid rgba(66, 153, 225, 0.3) !important;
  padding: 2px 8px !important;
  border-radius: 12px !important;
  font-size: 0.85em !important;
  font-weight: 500;
  text-decoration: none !important;
  transition: all 0.2s ease;
}

a.tag:hover,
.cm-hashtag:hover {
  background-color: var(--interactive-accent, #4299e1) !important;
  color: #ffffff !important;
  border-color: var(--interactive-accent, #4299e1) !important;
}
```

## 十三、阅读模式页面宽度控制 (`page-width-control.css`)

> 自定义控制笔记正文的可读行宽（可按需调大，让整篇笔记容纳更多内容）。

```css
/* 自定义笔记最大正文宽度 */
.markdown-source-view.mod-cm6 .cm-sizer,
.markdown-reading-view .markdown-preview-section {
  max-width: 900px !important; /* 默认通常是 700px-750px，可在此处增减 */
  margin: 0 auto !important;
}
```

## 🛠️ 文件划分标准与维护逻辑总结

1. **一类组件 = 一个 `.css` 文件**
    
    - 表格：`table-all-style.css`
        
    - 图片：`image-card.css`
        
    - 复选框：`checkbox-style.css`
        
        _好处_：出现样式异常时，可以在 **设置 -> 外观 -> CSS 代码片段** 中逐个关闭开关排查，而无需修改繁杂的代码。
        
2. **样式覆盖技巧**
    
    - 修改 CSS 保存后，可以在 Obsidian 中按下 `Ctrl + R` (或 `Cmd + R`) 快速重载全库样式。
        
    - 排序靠下的代码片段具有更高的优先级；若与第三方主题冲突，可以使用 CSS 变量（如 `var(--interactive-accent)`）来继承主题色。
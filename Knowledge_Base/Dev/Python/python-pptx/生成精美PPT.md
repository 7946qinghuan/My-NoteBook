
```plain text
模板文件 .pptx
└── SlideMaster（一个模板可能有多个）
    └── SlideLayout
        └── Layout Placeholder
            └── Slide Placeholder
```
每张幻灯片必须基于一个 Layout；Layout 再继承 Master。模板可能包含多个 Master，而 `prs.slide_layouts` 只是第一个 Master 的便捷入口。[python-pptx Slides API](https://python-pptx.readthedocs.io/en/stable/api/slides.html)

## 路径一：没有提供模板

这里不需要真正创建“空白母版”，而是：

```python
prs = Presentation()
blank_layout = 找到默认 Blank layout
slide = prs.slides.add_slide(blank_layout)
```

`Presentation()` 本身已经带有默认 Master 和多个 Layout。我们只是选择 Blank Layout，然后完全使用自己的组件和定位系统制作页面。

流程：

```plain text
Presentation()
    ↓
找到 Blank Layout
    ↓
添加空白 Slide
    ↓
使用组件函数创建标题、文本、图片、卡片、图表
    ↓
运行 SlideLayoutInspector
    ↓
保存
```

这里的 `DefaultLayout.BLANK = 6` 可以继续使用，因为它只针对 python-pptx 自带默认模板。

## 路径二：提供模板

不能再依赖固定 Layout 下标。官方文档明确说明，模板中的 Layout 数量和顺序都可能不同，固定索引只是默认主题的惯例。[python-pptx Working with Slides](https://python-pptx.readthedocs.io/en/latest/user/slides.html)

正确流程应是：

```plain text
加载模板
    ↓
遍历全部 SlideMaster
    ↓
遍历每个 Master 的 Layout
    ↓
分析 Layout 名称、元素和占位符
    ↓
建立 LayoutCatalog
    ↓
根据页面需求选择 Layout
    ↓
创建 Slide
    ↓
根据 placeholder idx 填充内容
    ↓
添加无法直接填入占位符的普通元素
    ↓
运行布局检查
```
# 仓库说明

这是一个 Obsidian 个人知识库 vault，用 `obsidian-git` 插件做同步。不是代码项目，没有构建/测试流程；对这个仓库的"开发"就是读写 Markdown 笔记。

## 规则在哪

**笔记编写、归类、双链的完整规则写在 `meta/conventions/` 下，那是唯一信息源。动笔前先读对应的那篇，不要凭这份摘要臆断：**

- `meta/conventions/目录与归类规则.md` — 一篇新笔记该放哪，`Knowledge_Base/` 的 7 个域怎么分
- `meta/conventions/笔记编写规范.md` — frontmatter 格式、文件命名禁忌、图片处理
- `meta/conventions/双链与-Graph-维护规则.md` — MOC 页怎么维护、`## 相关笔记` 小节怎么加

模板在 `meta/templates/`：`Daily-Note-Template.md`（每日笔记）、`Review-Template.md`（复盘）。

## 目录速览

```
Knowledge_Base/     可复用的主题参考笔记，固定 7 个域，最多两层
  AI-Engineering/ Dev/ Toolchain/ Embedded-System/ Projects/ Workplace/ Productivity/
Daily-Task-List/    每日笔记 YYYY/Month/YYYY-MM-DD.md；旧版月度清单在 Archive/（只读）
Plans/              跨月的个人学习/成长计划
Inbox/              未归类草稿，定期用 note-capture 清空
meta/               模板与本仓库自身的规则
images/             所有图片附件，全部平铺
```

## 几条容易踩的坑

- **文件名不要用方括号 `[]`** — 会打断 `[[wikilink]]` 语法导致链接失效，已经踩过一次。
- **新建笔记后要挂到所属域的 MOC 页**，否则它在 Graph 里是孤点。
- **归类前先查重** — 用 `vault-search` 或 grep 搜一下，别重复造轮子。
- **`.` 开头的目录 Obsidian 不显示**，所以给人看的内容不要放进 dot 目录。

## 可用的 Skills

- **`note-capture`** — 归档新内容：判断归类、补 frontmatter、查重、挂 MOC、建双链。
- **`vault-search`** — 用自然语言问题检索整个 vault 并综合作答，标注引用的笔记路径。
- **`vault-review`** — 生成一段时间范围的复盘草稿，落到 `Daily-Task-List/Reviews/`。

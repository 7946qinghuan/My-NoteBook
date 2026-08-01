# 仓库说明

这是一个 Obsidian 个人知识库 vault，用 `obsidian-git` 插件做同步。不是代码项目，没有构建/测试流程；对这个仓库的"开发"就是读写 Markdown 笔记。

## 目录结构与归类规则

- **`Knowledge_Base/`** — 可复用的主题参考笔记，按技术/领域分子目录（如 `RAG/`、`Linux/`、`Github/`、`WTO-IOTC/` 等）。判断标准：这篇笔记以后还会被检索、引用、复制粘贴用，内容与写作时间无关。新主题不存在时可以直接新建子目录，不要硬塞进不相关的现有主题。
- **`Daily-Task-List/YYYY/Month/YYYY-MM-DD.md`** — 每日笔记，一天一个文件（2026-08-01 起生效），记录"今天做了什么/要做什么"，时间线性质，不要把可复用的技术笔记堆在这里面。新建时用 `_templates/Daily-Note-Template.md`。
- **`Daily-Task-List/Archive/YYYY/`** — 2026-08-01 之前的旧版月度任务清单（`N月任务清单.md`，一个文件包含全月每天的内容），只读历史存档，不再新增内容。
- **`Plans/`** — 个人成长/学习类计划（例如英语学习计划、前端学习计划），跟当月任务清单不同，这类内容跨月甚至跨年有效。
- **`Inbox/`** — 还没想好归类、临时速记/草稿的笔记。定期应该被清空：内容成熟后用 `note-capture` skill 归档到 `Knowledge_Base/` 或 `Plans/`。
- **`Daily-Task-List/Reviews/YYYY/`** — 由 `vault-review` skill 生成的周期性复盘草稿。
- **`images/`** — 所有笔记的图片附件，全部平铺在一个目录里，靠 Obsidian 的全局文件名解析被引用，不要按笔记建子目录。
- **`_templates/`** — 笔记模板：`Daily-Note-Template.md`（每日笔记）、`Review-Template.md`（复盘）。

## 笔记约定

- 新建笔记默认加 frontmatter：
  ```yaml
  ---
  title: 笔记标题
  date: YYYY-MM-DD
  tags: [主题, 子主题]
  aliases: []
  ---
  ```
  这是**新笔记**的约定；vault 里已有的历史笔记大多没有 frontmatter，不需要为了统一格式去批量回填。
- 图片一律放进 `images/`，用 Obsidian 的 `[[文件名]]` 方式引用，不要建独立的图片子目录。
- 归类前先用 `vault-search` 或 grep 确认相关主题下是否已经有类似笔记，避免像"简历评分表"那样同一份内容散落多处、重复造轮子（`Knowledge_Base/Management/招聘评分表/` 就是一次事后合并的教训）。

## 可用的 Skills

- **`note-capture`** — 归档一段新内容/新笔记：判断放进 `Knowledge_Base` 的哪个主题（或新开主题）/ `Daily-Task-List` / `Plans` / `Inbox`，补 frontmatter，查重并建议双链。
- **`vault-search`** — 用自然语言问题在整个 vault 里检索并综合作答，回答时标注引用的笔记路径。
- **`vault-review`** — 生成一段时间范围（默认最近一周）的复盘草稿，汇总任务清单进展和新增笔记，落到 `Daily-Task-List/Reviews/`。

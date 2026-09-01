# 双路径渲染：传模板走确定性产线，不传模板走 agent 排版

## Context

当前 PPT 渲染的痛点：`VisualTemplate` 只能改画布、配色、字体、背景图/logo 挂在哪些版式上。
**内容区怎么排是全局唯一的一份** —— `LayoutEngine` 的 11 个 `_<archetype>()` 把矩形算死，
`PPTXRenderer` 把形状语言画死。想要第二种观感，只能改代码发版。

同事的 POC（`draft/jujun_ppt_poc/`）走 主题 → 大纲 → SVG → PPTX。探查结论：
**它没有样式系统，它是没有版式约束** —— `planner.py:26-41` 让模型编 5 个颜色，
`executor.py:39-45` 给 5 条散文提示，此外每个 `x/y/width/height` 都由 LLM 现场编。
代价是不可复现、无自动换行（`svg2pptx.py:200` 写死 `word_wrap=False`）、
越界只查文本锚点不查文本长度（`check_svg.py:135-149`）。
（值得肯定的一点：它的 pptx 是**原生可编辑**的，不是图片。）

**采纳的方案**：保留两套默认模板，`template` 传了就走现有确定性产线（一行不改），
不传就启动 agent，配一套组件化能力让它按内容自己排版。agent 是 opt-in，
现有的字节级可复现、40 个渲染测试、07/08 溢出标定全部不受影响。

## 核心设计决策：模型排版，但不算数

POC 的病根不是 SVG 这个格式，是**让模型决定坐标却不给它尺子**。
所以 agent 不产出绝对坐标，产出一份**相对版面 DSL**，算术由 Python 做：

```json
{
  "areas": [
    {"op": "rows", "id": "main", "parent": "page", "sizes": [1.05, "auto"], "gap": 0.10},
    {"op": "grid", "id": "cells", "parent": "main.1", "rows": 2, "cols": 2, "gap": 0.22}
  ],
  "placements": [
    {"op": "text",  "area": "main.0",  "ref": "title",    "role": "page_title"},
    {"op": "panel", "area": "cells.0", "shape": "left_bar", "fill": "surface"},
    {"op": "block", "area": "cells.0", "ref": "cards[0]", "container": "cells.0"}
  ]
}
```

这一个决定同时解决四件事：

1. **模型不做算术。** `(10.2272 - 2×0.56 - 0.22) / 2 = 4.4436` 这种题模型算不对，
   算错就是无穷重试。DSL 里它只说「横向切两份」。
2. **`check_layout()` 原样复用。** DSL 解析出来就是 `tuple[LayoutElement, ...]`，
   正是 `layout.py:659` 那个校验器的入参 —— 越界/重叠/子元素跑出容器，一行新代码都不用写。
3. **`fit_font_size()` 原样复用。** placement 带 `role`，role → (preferred, minimum)。
   字号仍由渲染端按框实测，模型碰不到。
4. **模型不能改文本。** placement 用 `ref` 指向已审改内容的字段路径，不重打文本。
   POC 的 SVG 里文本是模型重打的，会漂移；我们这里结构上不可能。

## 与「样式可定制」重构的关系：那是本方案的前置

之前盘的三阶段样式重构（形状语言 / 角色字号表 / 几何）在这里被吸收：

- `panel.shape` 的取值词表（`rounded` / `rect` / `left_bar` / `none`）**就是阶段一**
  （形状语言 + 把 `pptx.py` 里 12 个游离十六进制色收进设计层）。
- `text.role` → (preferred, minimum) 的查表**就是阶段二**
  （收掉 20 组内联字面量；顺带修掉段间距常量在估算侧 `pptx.py:813,940`
  与写入侧 `:839,843,853,918,954,959` 各写一遍、必须手工同步的隐患）。
- 阶段三（几何常量）**不需要做** —— agent 路径的几何来自 DSL，
  确定性路径的几何维持现状即可。

所以顺序是：先做设计层 + 角色表（不动几何，07/08 标定不受影响），再做 agent 路径。

## 参考实现：`draft/repositories/claude-pptx-skills`

同一问题的另一份实现（配套文章 `draft/ppt_design/article.md`），思路高度重合，
`scripts/grid.py:1-27` 的 docstring 与本方案的 DSL 出发点一致：
「deliberately *not* a catalogue of named slide layouts… this module only does
the arithmetic」。agent 只决定 arrangement，chrome（logo / 标题 / 页脚）固定不参与。

**采纳它三样东西：**

1. **三层设计层**（`design/brand.tokens.json`，180 个叶子）：
   `primitive`（唯一的字面值：色板 / 字体 / 字号阶 / 间距阶）
   → `semantic`（角色：`text.heading`、`chart.categorical`）
   → `type` / `layout` / `component` / `charts`（配方，用 `{alias}` 指回上两层）。
   我们的 `ThemeSpec` 只有**一层**，`primary` / `muted_text` 直接绑十六进制，
   既换不掉底层色板，也没有「`type.title` = 字体+字号+颜色」这种整体角色。
2. **设计层是外部 JSON，不是 Python dataclass。** 加一套风格不发版；
   更关键的是 Phase A 可以**直接产出一份 tokens**，与「不传 template 走 agent」天然契合。
   marine / default 各落一份 JSON，agent 路径以其中一份为基底做覆盖。
3. **conformance gate**（`scripts/check_design.py`，248 行）：打开生成好的 pptx，
   遍历每个 shape，**用了 tokens 未声明的字体或颜色就 FAIL**。
   这是我们完全没有的一类检查 —— 现有 `check_layout` 管几何、预览图靠目视，
   没有「产物只准用声明过的值」这一条。它把设计层从"建议"变成"拦得住的约束"。

**不采纳它一样东西（重要）：它没有任何字号自适应。**
`type.title.size` 写死 26pt，`build.py` 除 `word_wrap=True` 外没有 shrink/fit 逻辑，
溢出靠第三道人眼 render QA 兜（文章原文承认 "a shape can be technically valid and
still overlap something or run off the edge"）。这正是我们花三轮治好的病。
所以移植 token 时**必须改一处语义**：

> token 里的 `size` 是 **preferred**，不是最终值；仍然要过 `fit_font_size()` 按框实测。

同理 `grid.py` 的 `sizes=[1.05, 0.95, "auto"]`（"auto" 吸收余量）比我原来的纯 `ratios`
好用，直接采纳；但它的重叠检查是事后 WARN，我们的 `check_layout()` 在写 XML 之前就拦、
且多一条容器包含校验，两层都保留。

## 三个阶段的产线结构

```
PresentationContent（已生成、已校验的 11 种版式内容）
  ↓
Phase A  定设计（1 次调用，整份）      → DeckDesign
         画布 + 配色 + 字体 + 每种版式的排版意图（各一句）
  ↓                                    ← 没有这步，22 页会长成 22 个样子
Phase B  逐页排版（N 次，并发 5 路）   → PageLayoutDSL
         ask_structured(schema=PageLayoutDSL, validate=检查)
         不过就把 check_layout 的中文问题清单喂回去重试
  ↓
Phase C  编译（纯 Python，确定性）     → pptx bytes
         DSL → Rect 树 → LayoutElement → 复用现有绘制 helper
```

**Phase B 不需要 function-calling。** 用上个月刚落地的 `ask_structured(validate=...)`
（`services/utils/model_call.py:12`）就够：`validate` 钩子跑 `check_layout`，
异常消息就是问题清单，`ask_structured` 自己带错重试。这是 POC 那个重试循环的正确版本，
且机制已经在 `homework.py` / `_write_slide` 上跑通、有测试。

真正的 function-calling（模型中途调 `measure_text` 再决定）留到 v2，
**只在实测重试率偏高时才加** —— `camel.toolkits.FunctionTool` 可用，届时是加法不是改法。

**每页兜底**：重试用尽仍不过校验的页，回落到该版式的确定性排版
（`LayoutEngine.layout_slide`）+ agent 定的配色。`LayoutEngine` 现成，兜底几乎免费，
保证整份一定渲得出来。

## 接口改动（有一处破坏性变更，需要确认）

`schemas/request/ppt.py` 的 `PresentationRenderRequest.template` 目前是
`str = Field(default="marine")`，**「没传」和「传了 marine」现在无法区分**。
要按本方案分流，得改成 `str | None = Field(default=None)`。

副作用：现有样例里 **01 / 05 / 06 三份没传 `template`**，改完会从 marine 切到 agent 路径。
（07/08 显式传了 marine，不受影响，溢出标定安全。）

处理办法：这三份补上 `"template": "marine"` 保持原意，另加一份不传 template 的
agent 路径样例。端点、轮询、`TaskKind` 全不变。

## 代码落点

| 文件                                                                               | 动作                                                           | 量                         |
| ---------------------------------------------------------------------------------- | -------------------------------------------------------------- | -------------------------- |
| `components/render/assets/tokens/{marine,default}.json`                          | 新增两份三层设计层 JSON                                        | —                         |
| `components/render/tokens.py`                                                    | 新增：加载 +`{alias}` 解析 + Pydantic 校验                   | ~180                       |
| `components/render/dsl.py`                                                       | 新增 DSL 模型 + 解析器（DSL →`SlideLayout`）                | ~250                       |
| `components/render/conformance.py`                                               | 新增 conformance gate（产物只准用声明过的字体/颜色）           | ~150                       |
| `components/render/agent_layout.py`                                              | 新增 Phase A/B 驱动、重试、逐页兜底                            | ~200                       |
| `components/render/pptx.py`                                                      | 新增`_render_from_placements()`；helper 改吃解析后的 token   | ~150 改 + 现有 helper 调整 |
| `components/render/template.py`                                                  | `VisualTemplate` 指向 token 文件；`ThemeSpec` 由 JSON 取代 | ~40 改                     |
| `components/prompt/render_deck_design.mdcomponents/prompt/render_page_layout.md` | 新增两份提示词                                                 | —                         |
| `schemas/request/ppt.py`                                                         | `template: str \| None = None`                                | 1 行                       |
| `services/ppt.py:291`                                                            | 按`req.template` 是否为空分流                                | ~10                        |

复用（不新写）：`check_layout()` `layout.py:659`、`fit_font_size()` `layout.py:758`、
`LayoutEngine.layout_slide()` `layout.py:227`（兜底）、`ask_structured()` `model_call.py:12`、
`_add_shape/_add_text_box/_add_shape_text/_add_table` `pptx.py:1067-1195`、
`pptx_to_page_images()` `preview.py`。

## 建议的提交批次

一批 = 一个能独立跑通的阶段：

1. **设计层**：两份 tokens JSON + `tokens.py` 解析；`ThemeSpec` 由它取代，
   12 个游离十六进制色收进 primitive 层。形状词表进 `component` 层。几何不动。
2. **角色表**：`type.*` 层接上 `fit_font_size`（token 的 size 当 preferred），
   收掉 20 组内联字面量与重复的段间距常量。
3. **conformance gate**：`conformance.py` + 拿现有 8 份样例的产物做基线
   （**纯离线**，先当报告跑通，确认零误报再改成阻断）。
4. **DSL**：模型 + 解析器 + 接上 `check_layout`（**纯离线，不碰模型**，可单测）。
5. `_render_from_placements()` —— 手写一份 DSL 就能渲出 pptx。
6. Phase A/B 的 agent 驱动 + 提示词 + 逐页兜底。
7. 接口分流 + 样例迁移。

## 验证

- **回归（每批都跑）**：`uv run pytest tests/test_steins_agent/test_services/test_ppt_render.py`
  全绿；8 份样例全量重渲染，**只有 08 打 `仍装不下` 告警**，其余 0 条；
  `02`/`03` 的几何逐页与改动前一致（确定性路径不该有任何变化）。
- **批 3**：对现有 8 份样例的产物跑 conformance gate，**应当零 FAIL** ——
  有 FAIL 说明设计层漏收了值（正是它要抓的），补进 token 而不是放宽检查。
- **批 4 单测**：手写 DSL → 断言解析出的矩形、断言 `check_layout` 能抓出
  越界/重叠/子元素逃出容器（复用 `test_ppt_render.py:283,300` 的既有用例形状）。
- **批 5**：手写一份 22 页 DSL，渲染 + `pptx_to_page_images()` 目视。
- **批 6**：真实跑 `06_真实测试.json` 的内容走 agent 路径，检查
  (a) 22 页观感是否一致（Phase A 是否生效），(b) 兜底触发了几页，(c) 平均重试轮数，
  (d) conformance gate 是否零 FAIL。重试率高就上 v2 的 `measure_text` 工具。
- `uv run ruff check` + `pyright` 干净；`test_log_prefix_convention.py` 仍过。
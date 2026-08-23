---
title: NanoBot 44 周学习路线
create_at: 2026-08-23
update_at: 2026-08-23
tags: [AI, Agent, NanoBot, Roadmap]
aliases: [NanoBot Roadmap]
---

# NanoBot 44 周学习路线

## 路线原则

- 从一次 Turn 的纵向切片开始，而不是按目录顺序阅读。
- 每完成一层，都回到完整 Turn 中重新定位它。
- 核心代码逐行学习，同构适配器使用代表深读和全量矩阵。
- CLI 与 WebUI 放在最后，作为内部状态的显示与交互层学习。
- 每章都要产生可以验证的作品，而不是只产生阅读时长。

## 第 1 阶段：最小闭环（第 1～6 周）

### 第 00 章：固定版本、运行项目、建立观察基线

- 认识安装、配置、入口和测试命令。
- 固定源码 commit，记录环境，不修改核心行为。
- 成果：一份可复现的运行基线。

### 第 01 章：一次完整 Agent Turn

- 从 Channel 入站追踪到 Outbound 投递。
- 先把 Context、Provider、Tool、Session 当作黑盒。
- 成果：一张端到端 Turn 图和一份调用链表。

### 第 02 章：MessageBus 与事件契约

- 阅读 `bus/events.py`、`bus/queue.py` 和出站/进度事件。
- 理解生产者—消费者、解耦、排队与并发。
- 成果：手写最小异步消息总线。

### 第 03 章：AgentLoop

- 理解 session、workspace、runtime、command、dispatch 和 delivery 编排。
- 区分 Channel-facing turn 与 model-facing loop。
- 成果：能够解释任一入站消息走到哪个分支。

### 第 04 章：AgentRunner

- 理解模型请求、Tool Call、Observation、继续迭代与最终答案。
- 学习重试、空响应、长度恢复、并发工具、注入和最大轮次。
- 成果：从零写出简化 Runner 状态机。

### 第 05 章：Mini Nanobot

- 用 150～300 行 Python 组合 Bus、Runner、Provider Stub 和 Tool Registry。
- 成果：一个带两个 Tool、会话历史和终止条件的 Mini Agent。

## 第 7 周：复习一

- 不阅读新模块。
- 脱稿重画 Turn、Bus、Loop、Runner。
- 给 Mini Agent 制造一个终止条件错误并定位原因。

## 第 2 阶段：上下文与状态（第 8～13 周）

### 第 06 章：ContextBuilder

- 系统提示词、用户消息、图片附件、技能和工作区指令如何组合。
- 成果：列出一次请求中每段上下文的所有权和生命周期。

### 第 07 章：Templates、Skills 与 Runtime Context

- Jinja2 Prompt、渐进式 Skills、动态 Request Context。
- 成果：增加一段受控的运行时上下文并验证边界。

### 第 08 章：Session Key 与 JSONL 持久化

- Session 命名、缓存、元数据、fork、恢复与统一 Session。
- 成果：解析并解释一份真实 Session 文件。

### 第 09 章：Context Governance 与 Compaction

- Token 估算、重放预算、上下文裁剪、摘要和空闲压缩。
- 成果：构造一个超长会话并预测保留内容。

### 第 10 章：Memory 与 Dream

- Session 与长期 Memory 的区别。
- Dream 的历史聚合、写入原子性与崩溃一致性。
- 成果：画出 Session → History → Memory 的生命周期。

### 第 11 章：Hook、Streaming、Checkpoint 与 Turn Delivery

- 内容流、Reasoning 流、进度、文件编辑事件和中断恢复。
- 成果：记录一次 Turn 的事件时间线。

## 第 14 周：复习二

- 从空白解释“模型这一次看到了什么”。
- 做一个只读 Session/Context 检视器。

## 第 3 阶段：工具与边界（第 15～20 周）

### 第 12 章：Tool Contract、Schema、Registry 与 Loader

- Tool 是如何声明、发现、注册、校验和暴露给模型的。
- 成果：实现一个无副作用 Tool。

### 第 13 章：Filesystem、Apply Patch、Search 与 File State

- 路径解析、读写能力、Patch 语义和每 Session 文件状态。
- 成果：实现一个受工作区限制的文件分析 Tool。

### 第 14 章：Shell、Exec Session、取消与 Sandbox

- 短命令、长运行会话、PTY、超时、取消和 bwrap。
- 成果：画出命令从参数到子进程退出的完整状态机。

### 第 15 章：Web Search、Fetch 与 SSRF

- URL 解析、DNS/IP 防护、重定向、白名单和内容清洗。
- 成果：设计一组恶意 URL 测试用例。

### 第 16 章：MCP、Agent Plugin、Skills 与 CLI Apps

- 代码能力、外部工具和工作流知识的不同扩展边界。
- 成果：判断十个需求分别应放 Tool、Skill、MCP 还是 Plugin。

### 第 17 章：Subagent、Session Messaging 与 Runtime Control

- 子任务的创建、结果注入、并发、会话间消息与运行时控制。
- 成果：追踪两个并行 Subagent 的结果如何回到父 Turn。

## 第 4 阶段：长程能力（第 21～23 周）

### 第 18 章：Sustained Goal 与 Long Task

- Goal 状态、继续消息、预算、完成/阻塞与恢复。
- 成果：用状态机解释长程目标。

### 第 19 章：Cron、Trigger、Heartbeat 与 Automation Turn

- 调度、at-least-once、Session 绑定、忙碌排队和结果投递。
- 成果：完成一个可重复验证的定时任务。

### 第 20 章：图片、附件与音频转录

- 多模态输入、图像生成 Provider、媒体文件和转录注册表。
- 成果：追踪一个媒体消息的端到端生命周期。

## 第 24 周：复习三

- 将 Goal、Subagent、Cron、Trigger 统一描述为事件和状态机。
- 解释取消、重试和重复投递之间的差异。

## 第 5 阶段：模型运行时（第 25～28 周）

### 第 21 章：Config Schema、Loader 与 Watcher

- Pydantic、camelCase alias、`${VAR}`、配置保存和热更新。
- 成果：从配置字段追踪到运行时对象。

### 第 22 章：Model Preset、Runtime、Registry 与 Factory

- Provider 自动判断、Model Preset、Generation 参数和运行时切换。
- 成果：预测不同配置组合最终选择的 Provider。

### 第 23 章：Provider Base、OpenAI Compatible 与 Responses API

- 消息转换、Tool Call、Streaming、Usage 和协议状态。
- 成果：同一对话通过两个协议适配器运行。

### 第 24 章：Specialized Provider、OAuth、Fallback 与 Conversation State

- Anthropic、Azure、Bedrock、Codex、Copilot 等差异。
- 成果：完成 [[templates/provider-matrix]]。

## 第 6 阶段：输入输出与常驻运行（第 29～34 周）

### 第 25 章：BaseChannel 与事件契约

- 外部平台如何翻译为统一 Inbound/Outbound Message。
- 成果：实现一个内存 Channel。

### 第 26 章：Channel Registry、Manager 与 Outbound Delivery

- 自动发现、生命周期、流式合并、重试和错误隔离。
- 成果：解释单个 Channel 崩溃为何不应拖垮 Agent Core。

### 第 27 章：WebSocket Channel

- 鉴权、多 Chat 复用、Typed Envelope、Streaming 和 WebUI 请求。
- 成果：编写一个最小 WebSocket 客户端。

### 第 28 章：Bot SDK Channel 家族

- Telegram、Discord、Slack、QQ、NapCat 等代表实现。
- 成果：按共同骨架和平台差异分类。

### 第 29 章：企业、设备、联邦与邮件 Channel 家族

- Feishu、DingTalk、WeCom、Teams、Mattermost、Weixin、WhatsApp、Signal、
  Matrix、Email 等。
- 成果：完成 [[templates/channel-matrix]]。

### 第 30 章：Gateway、Process Runtime、Lease 与 Pairing

- Composition Root、前后台进程、客户端租约、启动关闭和访问审批。
- 成果：画出 Gateway 进程与 Channel/Automation 的生命周期。

## 第 35 周：复习四

- 随机选择一个 Channel，端到端追踪消息。
- 比较 Channel retry、Provider retry 和 Automation retry。

## 第 7 阶段：公开入口与显示层（第 36～41 周）

### 第 31 章：Python SDK 与 OpenAI-compatible API

- SDK Facade、Session/Memory Client、HTTP、SSE 和模型列表。
- 成果：同一个请求从 SDK 和 HTTP 发起。

### 第 32 章：Command Router 与 CLI

- Typer 入口、Onboarding、Gateway 命令、Slash Command 和 Terminal/TUI 边界。
- 成果：从 CLI 参数追踪到 Agent Turn。

### 第 33 章：WebUI Bootstrap 与 NanobotClient

- HTTP Bootstrap、Token、WebSocket 生命周期和请求/响应关联。
- 成果：画出浏览器建立连接的全过程。

### 第 34 章：Stream Projection、Session 与活动状态

- Delta、Reasoning、Tool Trace、文件编辑和 Turn End 如何投影成 UI 状态。
- 成果：用事件序列预测页面状态。

### 第 35 章：Components、Workbench 与 Settings

- React 组件边界、Hooks、设置表单、工作区控制与懒加载。
- 成果：完成一个小型可测试 UI 改动。

### 第 36 章：构建、打包、部署、测试与安全总审计

- WebUI 产物进入 Wheel、CI、类型检查、部署和跨边界安全。
- 成果：为一次跨层修改写出分层验证计划。

## 第 8 阶段：掌握验证（第 42～44 周）

### 第 37 章：架构内扩展

- 完成一个带测试、文档和安全说明的扩展能力。
- 要求：不把边缘能力硬塞进 Agent Loop。

### 第 38 章：跨层故障诊断

- 从用户现象开始，只依靠日志、测试和源码定位问题。
- 要求：先判断所有权，再进入具体实现。

### 第 39 章：架构答辩

- 脱稿讲解完整系统。
- 回答如何增加 Provider、Tool、Channel、Skill、MCP、自动化和 UI 事件。
- 写出“如果由我重新设计 NanoBot”的保留项与修改项。

## 最终完成标准

- 能从任意入口追踪一次 Turn。
- 能解释 Context、Session、Memory 三者的区别。
- 能安全地扩展 Tool、Provider 或 Channel。
- 能定位长任务、流式输出、恢复和重复投递问题。
- 能为修改选择正确的模块所有者和验证层级。
- 能说明 NanoBot 当前设计的优势、限制与演进方向。


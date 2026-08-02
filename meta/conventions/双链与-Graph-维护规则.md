---
title: 双链与 Graph 维护规则
date: 2026-08-02
tags: [meta, 规范]
aliases: [Graph规则, 双链规则]
---

# 双链与 Graph 维护规则

## 为什么需要这套规则

Graph 视图靠笔记之间的 `[[双链]]` 才有意义。重构前这个 vault 有 127 篇笔记但**笔记之间的双链数量是 0**，Graph 打开就是 127 个互不相连的孤点，完全没用。

更根本的原因：**文件夹只能给一篇笔记一个位置，但一篇笔记可以和很多篇相关。** 典型例子是 4 篇代理配置笔记——

| 笔记 | 位置 |
|---|---|
| Linux服务器挂靠Windows V2Ray代理 | `Toolchain/Linux/` |
| Mac_Terminal_配置_Clash_Verge_代理 | `Toolchain/Mac/` |
| VSCode_Remote_Multi_Device_Proxy_Switch | `Toolchain/VSCode/` |
| macOS_Headscale_Tailscale_接入 | `Toolchain/Network/` |

它们是同一类问题，但按操作系统分也对、按工具分也对，目录结构永远没法把它们放一起。**这就是必须靠链接解决的部分。**

## 两类链接结构

### 1. MOC 枢纽页

- 总索引：`Knowledge_Base/Knowledge-Base-MOC.md`
- 每个域一篇：`Knowledge_Base/<域>/<域>-MOC.md`

MOC 页列出该域下所有笔记的链接，是 Graph 里的中心节点。

> **新建笔记后必须把它加进对应的 MOC 页**，否则它在 Graph 里就是孤点。

### 2. 笔记间横向链接

相关笔记末尾用 `## 相关笔记` 小节互链，格式：

```markdown
---

## 相关笔记

**这一组笔记的共同点是什么**

- [[笔记A]]
- [[笔记B]]
```

> **双向都要加。** 在 A 里链了 B，也要去 B 里链回 A。虽然 Obsidian 有反向链接面板能看到单向链接，但 Graph 里的连线和阅读时的可发现性都依赖显式双向链接。

## 现有的聚类

已经建立横向链接的 12 组（新笔记如果属于其中一类，直接加进去）：

- 代理与远程访问
- RAG 处理链路（文档解析 → 切分 → 分词 → 向量检索）
- Agent 设计与 Prompt 工程
- Knowledge-Agent 提示词版本演进
- FastAPI 分层架构与异常处理
- STM32 HAL 库学习路径
- Git / GitHub 协作规范
- WTO 谈判智能体设计演进
- WTO 背景知识与语料收集
- 招聘评分表
- Obsidian 笔记工作流
- OpenMV 基础

另有 `Projects/WTO-IOTC/` 下 8 组"教学大纲 ↔ 知识点"的一一配对。

## Graph 视图建议设置

在 Graph 的 **Groups** 里按 tag 建颜色分组，结构会清晰很多：

- `tag:#MOC` → 醒目颜色（枢纽节点一眼可见）
- `tag:#AI`、`tag:#Dev`、`tag:#Toolchain`、`tag:#Embedded`、`tag:#Project` → 各自一色

## 相关笔记

- [[目录与归类规则]]
- [[笔记编写规范]]

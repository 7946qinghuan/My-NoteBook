---
title: NanoBot Channel 能力矩阵
create_at: 2026-08-23
update_at: 2026-08-23
tags: [AI, Agent, NanoBot, Channel]
aliases: [NanoBot Channel Matrix]
---

# NanoBot Channel 能力矩阵

## 阅读方法

先深读 `BaseChannel`、`ChannelManager` 和 WebSocket，再为每个 Channel 填表。
每个结论必须来自 runtime、manifest、validation 或测试，不凭平台常识填写。

| Channel | 连接方式 | 入站 | 出站 | Streaming | Reasoning | Media | 鉴权/Pairing | Retry | 关键文件/测试 |
|---|---|---|---|---|---|---|---|---|---|
| WebSocket |  |  |  |  |  |  |  |  |  |
| Telegram |  |  |  |  |  |  |  |  |  |
| Discord |  |  |  |  |  |  |  |  |  |
| Slack |  |  |  |  |  |  |  |  |  |
| Feishu |  |  |  |  |  |  |  |  |  |
| Weixin |  |  |  |  |  |  |  |  |  |
| WeCom |  |  |  |  |  |  |  |  |  |
| WhatsApp |  |  |  |  |  |  |  |  |  |
| QQ |  |  |  |  |  |  |  |  |  |
| NapCat |  |  |  |  |  |  |  |  |  |
| DingTalk |  |  |  |  |  |  |  |  |  |
| Matrix |  |  |  |  |  |  |  |  |  |
| Mattermost |  |  |  |  |  |  |  |  |  |
| MS Teams |  |  |  |  |  |  |  |  |  |
| Signal |  |  |  |  |  |  |  |  |  |
| Email |  |  |  |  |  |  |  |  |  |
| MoChat |  |  |  |  |  |  |  |  |  |

## Channel 深读记录

### Channel 名称

- 外部事件如何转换为 `InboundMessage`：
- `OutboundMessage` 如何转换为平台请求：
- 流式和消息合并策略：
- 媒体处理：
- 鉴权与允许列表：
- 重试和错误隔离：
- 平台特殊限制：
- 最接近的测试：


# A2A：智能体间互操作协议

> Sources: Google《Announcing the Agent2Agent Protocol》，2025-04-09；Linux 基金会 2025-06 接管，交叉印证
> Raw: [A2A（verbatim/中文）](../../raw/ai-engineering/2025-04-09-a2a-protocol.md)

## 概述

Google 2025-04-09 发布的开放协议，使不同厂商/框架构建的 AI 智能体能互相发现、安全通信、委派任务、协调行动。2025-06-23 捐赠给 Linux 基金会（创始成员含 AWS、Cisco、Google、Microsoft、Salesforce、SAP、ServiceNow）。

## 与 MCP 的关系：互补正交

| | MCP（Anthropic, 2024-11） | A2A（Google, 2025-04） |
|---|---|---|
| 连接 | Agent ↔ 工具/数据/上下文 | Agent ↔ Agent |
| 方向 | 纵向：给 agent 接能力 | 横向：让 agent 互协调 |

官方明确 A2A「complements MCP」，二者常组合使用。

## 核心概念

- **Agent Card**：JSON 能力广告，供发现"谁能干某任务"。
- **Tasks**：有生命周期的协议对象，支持即时/长时运行 + 状态同步。
- **Client / Remote Agent**：client 制定任务，remote 执行。
- **Capability Discovery**：公布能力供对端识别。

## 在工程化线中的位置

与 MCP 同属**协议标准化**阶段，解决多智能体互操作：把框架浪潮中"单系统内多 agent 协作"（AutoGen/CrewAI）提升为**跨厂商/跨框架标准协议**。MCP（接能力）+ A2A（连 agent）共同构成 Harness 等生产范式的接入基础设施。

## 相关条目

- [MCP: Model Context Protocol](model-context-protocol.md) — 同阶段互补双协议（接工具 vs 连 agent）。
- [2023–24 Agent 框架浪潮](agent-framework-wave.md) — AutoGen/CrewAI 的多 agent 协作被 A2A 标准化。
- [2024–25 能力跃迁](capability-leap.md) — 上游阶段。
- [Harness Engineering](harness-engineering.md) — 下游生产范式（终点）。

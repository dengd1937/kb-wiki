# MCP：连接 AI 与外部系统的开放标准

> Sources: Anthropic《Introducing the Model Context Protocol》，2024-11-25；modelcontextprotocol.io 官方文档；2026 现状经报道交叉印证
> Raw: [MCP（verbatim/中文/现状）](../../raw/ai-engineering/2024-11-25-model-context-protocol.md)

## 概述

Anthropic 2024-11-25 发布的开源标准，用于连接 AI 应用与外部数据源、工具、工作流。官方类比为「AI 应用的 USB-C 接口」——提供标准化连接方式。

## 解决的问题：M×N → M+N

此前每个 AI 应用对接每个工具/数据源都要定制集成（M×N 套胶水）。MCP 用一套通用协议把它降为 M+N：**构建一次，到处集成**。

## 架构

- 客户端-服务器模型：**MCP server** 暴露数据/工具；**MCP client/host**（Claude、ChatGPT、VS Code、Cursor 等）连接它们。
- 通信基于 **JSON-RPC**。
- 核心原语：**tools / resources / prompts**。
- Anthropic 同时开源规范、SDK、参考 server（Google Drive、Slack、GitHub、Git、Postgres、Puppeteer）。

## 现状（2026）

已成 Agent 互联互通**事实标准**；2026-04 月下载量约 11 亿；Anthropic/OpenAI/Block 联合捐赠给 Linux 基金会 Agentic AI Foundation。

## 在工程化线中的位置

承接能力跃迁：Agent 变强后，工具/上下文接入碎片化不可忍受 → MCP 把 Function Calling 的"单模型工具调用"升级为**跨应用标准化接入层**，是 SWE-agent ACI 思想的生态化标准化，也是后续设计模式收敛与 Harness 生产范式的基础设施前提。

## 相关条目

- [2024–25 能力跃迁](capability-leap.md) — 催生 MCP 的上一阶段（碎片化痛点）。
- [OpenAI Function Calling](../ai-agents/openai-function-calling.md) — 被 MCP 标准化、跨应用化的上游能力。
- [SWE-agent](swe-agent.md) — ACI「为 agent 设计接口」的思想近邻。
- [A2A: Agent2Agent Protocol](a2a-protocol.md) — 同阶段互补双协议（A2A 连 agent，MCP 接工具）。
- [Harness Engineering](harness-engineering.md) — 以标准化接入为前提的下游生产范式。

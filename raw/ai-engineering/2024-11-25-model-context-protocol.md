# Introducing the Model Context Protocol (MCP)

> Source（公告）: https://www.anthropic.com/news/model-context-protocol
> Source（规范/文档）: https://modelcontextprotocol.io/introduction
> Source（2026 现状，交叉印证）: 见下「现状」节
> Collected: 2026-05-18
> Published: 2024-11-25

## 元数据

- **发布方**：Anthropic
- **标题**：Introducing the Model Context Protocol
- **发布日期**：2024-11-25
- **性质**：开放标准 / 协议规范 + SDK + 参考实现（开源）

## 是什么

> "MCP (Model Context Protocol) is an open-source standard for connecting AI applications to external systems."

MCP 是连接 AI 应用与外部系统（数据源、工具、工作流）的开源标准。官方类比：

> "Think of MCP like a USB-C port for AI applications."

—— 像 AI 应用的「USB-C 接口」，提供连接外部系统的统一标准方式。

## 解决的问题：M×N 集成难题

此前每个 AI 应用要对接每个数据源/工具都需定制实现：M 个应用 × N 个数据源 = M×N 套胶水，难以规模化。MCP 用**一套通用协议**把它降为 M+N：构建一次，到处集成。

## 架构

- **客户端-服务器模型**：开发者要么通过 **MCP server** 暴露数据/工具，要么构建作为 **MCP client** 的 AI 应用去连接这些 server。
- **通信**：基于 **JSON-RPC**。
- 角色：MCP hosts（如 Claude/ChatGPT 等 AI 应用）、clients、servers、transports。
- 核心原语：**tools（工具/动作）、resources（数据）、prompts（工作流/特化提示）**。

## Anthropic 开源了什么

- MCP 规范与多语言 SDK
- Claude Desktop 的本地 MCP server 支持
- 预构建参考 server 仓库：Google Drive、Slack、GitHub、Git、Postgres、Puppeteer

## 早期采用者

- **Block**、**Apollo**（集成进自身系统）
- 开发工具：**Zed、Replit、Codeium、Sourcegraph**

> "Open technologies like the Model Context Protocol are the bridges that connect AI to real-world applications." — Dhanji R. Prasanna, CTO, Block

## 现状（2026，交叉印证）

据 2026 年报道：MCP 已成为 AI Agent 互联互通的**事实标准**；2026-04 月下载量突破约 11 亿次；Anthropic、OpenAI、Block 联合将其捐赠给 Linux 基金会下的 Agentic AI Foundation。生态广泛支持：Claude、ChatGPT、VS Code、Cursor 等。（来源：shengyayun.com tech-judgment 2026-04-26 等报道，非一手公告）

## 在工程化线中的位置

承接「能力跃迁」：Agent 能自主写代码/操作计算机后，**工具/上下文接入的碎片化（每 agent 一套接口胶水）**变得不可忍受——MCP 把 Function Calling 的"单模型工具调用"升级为**跨应用/跨工具的标准化接入层**。它是 SWE-agent「为 agent 设计接口（ACI）」思想的生态化、标准化，也是后续设计模式收敛（Building Effective Agents）与生产范式（Harness）的基础设施前提。

# Announcing the Agent2Agent (A2A) Protocol

> Source（公告）: https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/
> Source（捐赠 Linux 基金会，交叉印证）: https://www.linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-...
> Collected: 2026-05-18
> Published: 2025-04-09

## 元数据

- **发布方**：Google（Google Cloud）
- **标题**：Announcing the Agent2Agent Protocol (A2A) — A new era of agent interoperability
- **发布日期**：2025-04-09
- **后续**：2025-06-23 由 Google 捐赠给 Linux 基金会（成立 Agent2Agent 项目；创始成员含 AWS、Cisco、Google、Microsoft、Salesforce、SAP、ServiceNow）
- **性质**：开放协议规范 + 参考实现

## 是什么

A2A 是一个开放协议，使 AI 智能体能够**互相通信、安全交换信息、跨企业平台与应用协调行动——无论由哪个厂商或框架构建**。

## 解决的问题

企业部署自主智能体时，需要它们跨孤立数据系统协作。A2A 让**多智能体生态无缝互操作**，提升自主性、倍增生产力、降低长期成本。

> "Enabling agents to interoperate with each other, even if they were built by different vendors or in a different framework, will increase autonomy and multiply productivity gains."

## 与 MCP 的关系（互补，非竞争）

> A2A "complements Anthropic's Model Context Protocol (MCP), which provides helpful tools and context to agents."

- **MCP**：Agent ↔ 工具/数据/上下文（纵向：给单个 agent 接能力）
- **A2A**：Agent ↔ Agent（横向：让不同 agent 互发现、互委派、互协调）

二者正交，常组合使用。

## 核心概念

- **Agent Card**：JSON 格式的能力广告，使 agent 能发现"哪个对端能执行某任务"。
- **Tasks**：协议定义的对象，有生命周期；可立即完成，也支持长时运行 + 状态同步。
- **Client / Remote Agent 模型**：client agent 制定任务，remote agent 执行。
- **Capability Discovery**：agent 公布能力以供对端识别。

## 启动伙伴（50+，后增至 150+）

平台/技术：Atlassian、Box、Cohere、Intuit、LangChain、MongoDB、PayPal、Salesforce、SAP、ServiceNow、Workday 等；
服务商：Accenture、BCG、Capgemini、Cognizant、Deloitte、Infosys、KPMG、McKinsey、PwC、TCS、Wipro 等。

## 在工程化线中的位置

与 MCP 同属 Function Calling 之后的**协议标准化**阶段，但解决的是**多智能体互操作**：把框架浪潮中 AutoGen/CrewAI 等"单系统内多 agent 协作"提升为**跨厂商/跨框架的标准化 agent 间协议**。MCP（接能力）+ A2A（连 agent）共同构成 Harness 等生产范式所依赖的接入基础设施。

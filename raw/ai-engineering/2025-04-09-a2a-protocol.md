# A2A 协议详解（What is Agent2Agent (A2A)?）

> Source: https://www.ibm.com/think/topics/agent2agent-protocol
> Collected: 2026-05-21
> Published: Unknown
> Full text: https://www.ibm.com/think/topics/agent2agent-protocol

## 文章信息

- **作者**：IBM Think
- **载体**：IBM Think
- **发布日期**：Unknown
- **性质**：综述博客

---

Agent2Agent (A2A) 协议是一种用于人工智能 (AI) agent 的通信协议，最初由 Google 于 2025 年 4 月推出。该开放协议专为多 agent 系统设计，允许来自不同供应商或使用不同 AI agent 框架构建的 AI agent 之间实现互操作。

A2A 是一种 AI agent 通信的开放标准，类似于 IBM BeeAI 推出的 Agent Communication Protocol (ACP)。虽然早期的 agent 编排框架（如 crewAI 和 LangChain）在各自生态系统中自动化多 agent 工作流，但 A2A 协议充当一个消息层，让这些 agent 尽管具有不同的 agentic 架构，仍能相互"对话"。

可以将 A2A 想象为 agent 生态系统的通用语言或通用翻译器。它旨在打破孤岛，增强 agent 互操作性。

A2A 协议最初由 Google 及其他技术合作伙伴在 Google Cloud 平台下于 2025 年 4 月推出。1 它现在由 Linux Foundation 托管，作为开源 Agent2Agent (A2A) 项目。2

此前由 Anthropic 于 2024 年推出的 Model Context Protocol (MCP) 充当 AI 应用程序与外部服务（如 API（应用程序编程接口）、数据源、预定义函数和其他工具）有效通信的标准化层。与此同时，A2A 协议专注于 agent 协作，促进 AI agent 之间的通信。

这两个协议旨在相互补充。例如，一家零售商店可能拥有自己的库存 agent，该 agent 使用 MCP 与存储产品和库存水平信息的数据库进行交互。如果库存 agent 检测到产品库存不足，它会通知一个内部订单 agent，然后该订单 agent 使用 A2A 与外部供应商 agent 通信并下单。

## A2A 构建块

Agent2Agent 协议由多个用于 agent 交互的构建块组成：

* A2A client（客户端 agent）
* A2A server（远程 agent）
* Agent card
* Task
* Message
* Artifact
* Part

### A2A client（客户端 agent）

A2A client，也称为客户端 agent，可以是应用程序、服务或其他将请求委托给远程 agent 的 AI agent。它使用 Agent2Agent 协议发起通信。

### A2A server（远程 agent）

A2A server，也称为远程 agent，接收请求、处理任务并以状态更新或结果进行响应。它暴露一个与 Agent2Agent 协议兼容的 HTTP 端点。

### Agent card

这个 JSON 文件概述了 agentic AI 元数据，可以通过 URL 访问。它包含有关 agent 的基本信息，包括其名称、描述、版本、服务端点 URL、支持的模态或数据类型以及认证要求。

Agent card 类似于大型语言模型 (LLM) 的 model card。它们还宣传 agent 的能力和技能，充当名片、简历或 LinkedIn 个人资料，使 agent 能够相互发现。

### Task

一个 Task 表示完成一个请求所需的工作单元。它具有唯一 ID，并经历一系列定义的状态生命周期（submitted、working、input-required、completed、failed）。Task 对于多轮处理或长时间运行的 agent 间协作非常有用。

### Message

作为通信的基本单元，一条 Message 描述了对话中的一次交换或一个回合。它包含一个或多个承载实际内容的 Part。

Message 传递答案、上下文、指令、提示、问题、回复和状态更新。根据发送者的不同，每条消息都有一个关联的角色，可以是 agent role（用于服务器发送的消息）或 user role（用于客户端发送的消息）。

### Artifact

一个 Artifact 是 A2A server 作为其工作结果生成的有形产物。它可以是文档、图像、电子表格或任何其他可交付成果。与 Message 一样，Artifact 由一个或多个 Part 组成，并且可以增量流式传输。

### Part

一个 Part 是 Message 或 Artifact 中的一段内容。Part 根据其携带的数据有不同的类型。TextPart 是文本的容器，FilePart 表示文件，DataPart 包含结构化的 JSON (JavaScript Object Notation) 数据。

## A2A 如何工作

A2A 协议遵循客户端-服务器模型设置，具有三步工作流：

1. Discovery（发现）
2. Authentication（认证）
3. Communication（通信）

### Discovery

A2A 工作流始于一个实体（人类用户或其他 AI agent）向客户端 agent 发起请求。例如，用户可能请求帮助安排旅行，或者一个 AI agent 为零售商店的库存不足物品下订单。

然后客户端 agent 进行发现过程，查找远程 agent 并获取其 Agent card，以确定最适合执行任务的 agent。

### Authentication

一旦客户端 agent 识别出能够完成指定任务的远程 agent，它便根据 Agent card 中指示的安全方案进行认证。A2A 支持与 OpenAPI 规范对齐的安全方案，例如 API key、OAuth 2.0 和 OpenID Connect Discovery。

当客户端 agent 成功认证后，远程 agent 负责授权和授予访问控制权限。

### Communication

通信始于客户端 agent 向所选的远程 agent 发送 Task。Agent 间通信通过 HTTPS 进行安全传输，使用 JSON-RPC (Remote Procedure Call) 2.0 作为数据交换格式。

然后远程 agent 处理该 Task。如果它需要更多信息，它会通知客户端 agent 请求额外细节。一旦完成 Task，远程 agent 向客户端 agent 发送消息以及任何生成的 Artifact。

A2A 还为无法立即完成的更复杂 Task 提供任务管理功能，例如那些需要人工干预或涉及多个步骤的 Task。对于需要数小时或数天的长时间运行 Task，或者客户端 agent 断开连接的情况，A2A 协议允许通过发送到客户端提供的安全 webhook 的推送通知进行异步更新。对于大型或长时间输出或持续状态更新，A2A 协议使用 server-sent events (SSE) 支持实时流式传输。

## A2A 协议的优势

Agent2Agent 协议为现实世界 AI 系统中的 agent 通信提供了以下优势：

* Privacy（隐私）
* Seamless integration（无缝集成）
* Security（安全性）

### Privacy

该协议将 agentic AI 视为不透明的 agent。这种不透明性意味着自主 agent 可以在协作时无需暴露其内部运作方式，例如内部记忆、专有逻辑或特定工具实现。这有助于保护数据隐私和知识产权。

### Seamless integration

A2A 建立在现有标准之上，包括 HTTP、JSON-RPC 和 SSE。这使得企业更容易采用该协议，并有助于确保与当前技术栈的兼容性。

### Security

Agent2Agent 协议在设计时就考虑了安全性。它支持企业级认证和授权机制，并允许安全的信息交换。

## 未来路线图

A2A 仍处于早期阶段，因此组织可以预期随着协议的成熟会有改进。这些改进包括：在 Agent card 中正式纳入授权方案和可选凭证、动态检查意外或不支持技能的方法、在 Task 中支持动态用户体验 (UX) 协商（例如在对话过程中添加音频或视频），以及增强推送通知方法和流式传输可靠性。

有关更多信息，请访问 A2A 官方网站了解关键概念、深入了解协议规范、探索 Python 教程并下载软件开发工具包 (SDK)。A2A 还有一个 GitHub 网站提供代码示例和演示。

# Knowledge Base Index

## ai-agents

LLM-based agents 的基础研究：推理、规划、工具使用、自我反思等核心能力。

| Title | Summary | Collected |
|-------|---------|-----------|
| [Chain-of-Thought Prompting](ai-agents/2022-01-28-chain-of-thought.md) | 思维链提示激发 LLM 推理能力，大模型涌现出逐步推理能力 | 2026-05-19 |
| [ReAct](ai-agents/2022-10-06-react-reasoning-acting.md) | 协同推理与行动，将思考-行动-观察循环引入 LLM agent | 2026-05-19 |
| [Algorithm Distillation](ai-agents/2022-10-25-algorithm-distillation.md) | 上下文内强化学习，用 RL 历史序列训练 in-context 学习能力 | 2026-05-19 |
| [Chain of Hindsight](ai-agents/2023-02-06-chain-of-hindsight.md) | 后见之明链，用带标注的反馈序列对齐语言模型 | 2026-05-19 |
| [Toolformer](ai-agents/2023-02-09-toolformer.md) | 语言模型自学使用工具，自监督插入 API 调用 | 2026-05-19 |
| [Reflexion](ai-agents/2023-03-20-reflexion.md) | 言语强化学习，agent 通过自然语言自我反思改进决策 | 2026-05-19 |
| [LLM+P](ai-agents/2023-04-22-llm-p.md) | 用经典规划器赋能 LLM 最优规划能力，LLM 翻译 + PDDL 求解 | 2026-05-19 |
| [Tree of Thoughts](ai-agents/2023-05-17-tree-of-thoughts.md) | 思维树审慎搜索，在推理空间中系统性探索多条路径 | 2026-05-19 |
| [OpenAI Function Calling](ai-agents/2023-06-13-openai-function-calling.md) | 结构化工具调用从 prompt 进入模型原生能力，ReAct 的工程拐点 | 2026-05-18 |
| [LLM Powered Autonomous Agents](ai-agents/2023-06-23-llm-powered-autonomous-agents.md) | Lilian Weng 综述：planning / memory / tool use 三大组件全景 | 2026-05-19 |

## ai-engineering

AI agent 的工程化实践：框架、评测、协议、设计模式、生产范式。

| Title | Summary | Collected |
|-------|---------|-----------|
| [AutoGen](ai-engineering/2023-08-16-autogen.md) | 微软多智能体对话框架，可编程的 agent 间协作范式 | 2026-05-19 |
| [SWE-bench](ai-engineering/2023-10-10-swe-bench.md) | 自动化软件工程评测基准，用真实 GitHub Issue 衡量 LM 修 bug 能力 | 2026-05-19 |
| [SWE-agent](ai-engineering/2024-05-06-swe-agent.md) | 智能体-计算机接口（ACI），为 agent 设计专用交互环境 | 2026-05-19 |
| [MCP](ai-engineering/2024-11-25-model-context-protocol.md) | Anthropic 开放标准，AI 应用的 USB-C——标准化连接工具与数据源 | 2026-05-21 |
| [Building Effective Agents](ai-engineering/2024-12-19-building-effective-agents.md) | Anthropic 工程指南：克制设计，五种工作流模式 + 三条核心原则 | 2026-05-19 |
| [A2A Protocol](ai-engineering/2025-04-09-a2a-protocol.md) | Google Agent2Agent 开放协议，agent 间发现、认证、通信标准 | 2026-05-21 |
| [Harness Engineering](ai-engineering/2026-01-28-harness-engineering.md) | OpenAI 案例：零人工代码构建产品，为 agent 设计 harness 的生产范式 | 2026-05-14 |
| [Claude Code SDK & HaaS](ai-engineering/2025-09-23-claude-code-sdk-haas.md) | LLM API → Harness API 演进，Claude Code SDK 四维定制（prompt/tools/context/subagents） | 2026-05-21 |
| [Effective Harnesses (Long-Running)](ai-engineering/2025-09-30-effective-harnesses-long-running-agents.md) | Anthropic initializer + coding agent 双 agent 方案，跨 context window 增量推进 | 2026-05-21 |
| [Agent Frameworks/Runtimes/Harnesses](ai-engineering/2025-10-25-agent-frameworks-runtimes-harnesses.md) | LangChain 定义三层概念：Framework（抽象）→ Runtime（基础设施）→ Harness（开箱即用） | 2026-05-21 |
| [Deep Agents Harness Engineering](ai-engineering/2026-02-17-deep-agents-harness-engineering.md) | LangChain 实战：self-verification + tracing + context 注入，Terminal Bench 52.8→66.5 | 2026-05-21 |
| [Harness Design (Long-Running Apps)](ai-engineering/2026-05-15-harness-design-long-running-apps.md) | Anthropic 三 agent 架构（planner/generator/evaluator），GAN 启发的主观质量可分级 | 2026-05-21 |
| [Thariq HTML Effectiveness](ai-engineering/2026-05-08-thariq-html-effectiveness.md) | Claude Code 实践：用 HTML 作为 LLM 输出格式的惊人有效性 | 2026-05-18 |
| [Karpathy Ask for HTML](ai-engineering/2026-05-11-karpathy-ask-for-html.md) | Karpathy 观点：让 LLM 输出 HTML 结构化内容，重新定义人机 I/O 模态 | 2026-05-18 |
| [DeepAgents 官方文档](ai-engineering/deepagents/index.md) | LangChain DeepAgents 完整文档摄取（13 篇）：概念层 + Get Started + API Reference | 2026-05-24 |

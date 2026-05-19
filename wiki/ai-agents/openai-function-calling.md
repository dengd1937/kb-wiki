# OpenAI Function Calling：ReAct 条线的工程分水岭

> Sources: OpenAI《Function calling and other API updates》，2023-06-13；InfoQ 交叉印证
> Raw: [原文要点 / 来源说明](../../raw/ai-agents/2023-06-13-openai-function-calling.md)

## 概述

2023-06-13，OpenAI 随 `gpt-4-0613` / `gpt-3.5-turbo-0613` 引入 **function calling**：开发者用 JSON schema 描述一组函数，模型据输入**自行判断是否调用、调哪个、并产出符合签名的 JSON 参数**。模型经微调以可靠检测调用时机并返回合规 JSON。

## 核心机制

`描述函数(schema) → 模型决策 → 返回结构化 JSON 参数 → 应用执行 → 结果回灌` —— 把工具调用从「自由文本 + 脆弱解析」变为**原生、结构化、经微调保证格式**的能力。

## 官方用例

- 调用外部工具的聊天机器人；
- 自然语言 → API：`send_email(to, body)`；
- 结构化抽取：`get_current_weather(location, unit)`。

## 在 ReAct 条线中的意义

| | ReAct（2022） | Function Calling（2023.06） |
|---|---|---|
| 工具调用 | 提示引导、自由文本、需文本解析 | 模型原生、结构化 JSON、微调保证合规 |
| 可靠性 | 脆弱、易漂移 | 生产级 |

这是 Agent 从 demo 走向生产的**工程拐点**：把 Toolformer「工具使用进入模型能力」的思想产品化，Anthropic/Google 随后跟进，并成为后续 MCP、Agent Harness 的基础设施前提。

## 相关条目

- [ReAct](react-reasoning-acting.md) — 工具调用的提示时前身。
- [Toolformer](toolformer.md) — 工具能力进权重的思想，被本节点产品化。
- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — 其 Tool use 组件的工程落地。
- [Harness Engineering](../ai-engineering/harness-engineering.md) — 以可靠工具调用为前提的下游工程范式。

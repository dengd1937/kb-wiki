# AutoGen：多智能体对话框架

> Sources: arXiv 2308.08155（Wu et al., 微软研究院），2023-08-16
> Raw: [AutoGen（要点/中文）](../../raw/ai-engineering/2023-08-16-autogen.md)

## 概述

微软研究院 2023 年 8 月提出的开源框架，使开发者通过**多个可对话、可定制的智能体**构建 LLM 应用。智能体由 LLM、人类输入与工具的组合驱动，交互模式可用自然语言或代码声明式定义。覆盖数学、编码、问答、运筹、在线决策等领域。

## 核心思想

把"单 Agent 调工具"升级为**"多 Agent 通过对话协作"**：

- **Conversable Agents**：可收发消息、维护状态，由 LLM/人类/工具任意组合驱动。
- **可编程对话**：谁与谁对话、何时终止、是否引入人类，均声明式定义。
- 内置 UserProxyAgent / AssistantAgent、代码执行、GroupChat、人类在环。

## 在工程化线中的位置

属 Function Calling 之后的**框架浪潮**阶段，是"多 Agent 协作"工程范式的代表，亦为后续 Planner/Executor/Critic 角色分工的先声。

## 相关条目

- [2023–24 Agent 框架浪潮](agent-framework-wave.md) — 本条目所属阶段的综述。
- [OpenAI Function Calling](../ai-agents/openai-function-calling.md) — 多智能体协作的可靠工具调用前提。
- [Harness Engineering](harness-engineering.md) — 工程化线终点。

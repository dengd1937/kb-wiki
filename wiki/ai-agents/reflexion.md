# Reflexion：用语言反馈强化的语言智能体

> Sources: arXiv 2303.11366（Shinn et al., 东北大学/MIT/普林斯顿），2023-03-20；NeurIPS 2023
> Raw: [Reflexion（原文/中文）](../../raw/ai-agents/2023-03-20-reflexion.md)

## 概述

Reflexion 由 Shinn 等于 2023 年 3 月提出（作者含 ReAct 作者 Karthik Narasimhan、Shunyu Yao）。它解决"语言智能体如何快速从试错中学习"的问题——传统强化学习需大量样本和昂贵微调。Reflexion **不更新权重**，而是通过**语言反馈 + 情景记忆**强化智能体：对反馈进行口头反思，存入记忆缓冲区，指导后续试验做出更优决策。在编码基准 HumanEval 上达到 91% pass@1，超过当时 SOTA GPT-4 的 80%。

## 核心机制

- **Actor**：执行任务（常基于 ReAct）。
- **Evaluator**：给出反馈信号（标量或自由文本；外部或自我模拟）。
- **Self-Reflection**：把失败转化为具体口头反思（错在哪、下次怎么做）。
- **Episodic Memory**：反思存入记忆，下一轮注入上下文。

循环：`试验 → 评估 → 反思 → 记忆 → 重试`，实现无需微调的快速试错学习。

## 与 ReAct 的关系

Reflexion 直接构建在 ReAct 之上：ReAct 提供单轮 Thought–Action–Observation 循环，Reflexion 在外层加上**跨轮次的自我反思与记忆**，补齐了 ReAct 不会系统性地从失败中学习的短板。

## 关键结果

- HumanEval（编码）：**91% pass@1**，超过 GPT-4 的 80%。
- 序列决策、编码、语言推理多类任务均显著优于基线。
- 含反馈信号 / 融入方式 / 智能体类型的消融分析。

## 意义

确立"语言反馈 + 记忆"实现智能体自我改进的范式，是 Agent 记忆与自我纠错方向的奠基工作，影响后续 Self-Refine、记忆架构及多智能体 Critic 角色设计。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — Lilian Weng 的统一框架，Reflexion 属其自我反思 + Memory 的代表。
- [ReAct: Synergizing Reasoning and Acting in Language Models](react-reasoning-acting.md) — Reflexion 的底座推理-行动循环。
- [Toolformer: Language Models Can Teach Themselves to Use Tools](toolformer.md) — 同期工具使用方向。

# Algorithm Distillation：上下文内强化学习

> Sources: arXiv 2210.14215（Laskin et al., DeepMind），2022-10-25；ICLR 2023
> Raw: [Algorithm Distillation（要点/中文）](../../raw/ai-agents/2022-10-25-algorithm-distillation.md)

## 概述

AD（2022 年 10 月，DeepMind）用因果序列模型对 RL 算法的**训练历史**建模，把"学习算法本身"蒸馏进网络。将 RL 视为跨 episode 的序列预测问题——在源算法生成的数据上训练 Transformer，使其在**上下文中改进策略而不更新权重**，并能得到比源算法更数据高效的 RL。

## 核心思想

不蒸馏策略，而蒸馏"如何学习"：让模型读入"智能体逐步变强的完整学习史"，推理时仅靠上下文持续自我改进。

## 在 Lilian Weng 框架中的位置

**Planning → Self-Reflection**：被引为"通过历史轨迹实现上下文内自我改进"的代表，与 Reflexion 的"语言反思 + 记忆"形成不同技术路线对照。

## 意义

确立"in-context RL / 把学习算法序列化"范式，是上下文内自我改进与记忆驱动智能体的重要思想来源。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — 其 Self-Reflection 中"轨迹驱动自我改进"的代表。
- [Reflexion](reflexion.md) — 自我改进的对照路线（语言反馈 vs 历史蒸馏）。

# Reflexion: Language Agents with Verbal Reinforcement Learning

> Source: https://arxiv.org/abs/2303.11366
> Collected: 2026-05-18
> Published: 2023-03-20（arXiv v1；v4 2023-10-10）；NeurIPS 2023 接收

## 元数据

- **标题**：Reflexion: Language Agents with Verbal Reinforcement Learning
- **作者**：Noah Shinn、Federico Cassano、Edward Berman、Ashwin Gopinath、Karthik Narasimhan、Shunyu Yao
- **机构**：东北大学、MIT、普林斯顿大学（Karthik Narasimhan、Shunyu Yao 同为 ReAct 作者）
- **arXiv 编号**：2303.11366
- **版本历史**：v1 2023-03-20；v2 2023-05-21；v3 2023-06-10；v4 2023-10-10
- **会议**：NeurIPS 2023

## 摘要（原文）

Large language models (LLMs) have been increasingly used to interact with external environments (e.g., games, compilers, APIs) as goal-driven agents. However, it remains challenging for these language agents to quickly and efficiently learn from trial-and-error as traditional reinforcement learning methods require extensive training samples and expensive model fine-tuning. We propose Reflexion, a novel framework to reinforce language agents not by updating weights, but instead through linguistic feedback. Concretely, Reflexion agents verbally reflect on task feedback signals, then maintain their own reflective text in an episodic memory buffer to induce better decision-making in subsequent trials. Reflexion is flexible enough to incorporate various types (scalar values or free-form language) and sources (external or internally simulated) of feedback signals, and obtains significant improvements over a baseline agent across diverse tasks (sequential decision-making, coding, language reasoning). For example, Reflexion achieves a 91% pass@1 accuracy on the HumanEval coding benchmark, surpassing the previous state-of-the-art GPT-4 that achieves 80%. We also conduct ablation and analysis studies using different feedback signals, feedback incorporation methods, and agent types, and provide insights into how they affect performance.

## 摘要（中文翻译）

大语言模型（LLM）越来越多地作为目标驱动的智能体与外部环境（游戏、编译器、API）交互。然而，让语言智能体快速高效地从试错中学习仍很困难——传统强化学习需要大量样本和昂贵的模型微调。我们提出 **Reflexion**：一种不通过更新权重、而是通过**语言反馈**来强化语言智能体的新框架。具体而言，Reflexion 智能体对任务反馈信号进行**口头反思**，并把这些反思文本保存在**情景记忆缓冲区**中，以改进后续试验的决策。Reflexion 足够灵活，可纳入多种类型（标量值或自由文本）和来源（外部或内部模拟）的反馈信号，在多类任务（序列决策、编码、语言推理）上显著优于基线智能体。例如，在 HumanEval 编码基准上达到 **91% pass@1**，超过此前 SOTA GPT-4 的 80%。我们还就不同反馈信号、反馈融入方式和智能体类型做了消融与分析。

## 核心思想

Reflexion = 用"语言"做强化学习，**不改权重**：

1. **Actor**：执行任务的智能体（常基于 ReAct）。
2. **Evaluator**：对轨迹给出反馈信号（标量分数或自由文本，可来自环境或自我模拟）。
3. **Self-Reflection**：把失败/反馈转化为具体的口头反思（"我哪里错了、下次该怎么做"）。
4. **Episodic Memory**：将反思存入记忆缓冲区，在下一轮试验时注入上下文，指导更优决策。

通过"试验 → 反思 → 记忆 → 重试"的循环，实现无需微调的快速试错学习。

## 与 ReAct 的关系

Reflexion 直接构建在 ReAct 之上：ReAct 提供单轮 Thought–Action–Observation 推理-行动循环，Reflexion 在其外层加上**跨轮次的自我反思与记忆**，补齐了 ReAct"不会系统性地从失败中学习"的短板。两篇论文共享作者（Karthik Narasimhan、Shunyu Yao）。

## 关键结果

- **HumanEval（编码）**：91% pass@1，超过 GPT-4 的 80%。
- 在序列决策、编码、语言推理多类任务上均显著优于基线。
- 反馈信号、融入方式、智能体类型的消融分析。

## 意义

确立了"用语言反馈 + 记忆"实现智能体自我改进的范式，是 Agent 记忆与自我纠错方向的奠基工作之一，深刻影响了后续 Self-Refine、记忆架构与多智能体 Critic 角色设计。

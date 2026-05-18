# In-context Reinforcement Learning with Algorithm Distillation

> Source: https://arxiv.org/abs/2210.14215
> Collected: 2026-05-18
> Published: 2022-10-25（arXiv v1）

## 元数据

- **标题**：In-context Reinforcement Learning with Algorithm Distillation
- **作者**：Michael Laskin、Luyu Wang、Junhyuk Oh、Emilio Parisotto、Stephen Spencer、Richie Steigerwald、DJ Strouse、Steven Hansen、Angelos Filos、Ethan Brooks、Maxime Gazeau、Himanshu Sahni、Satinder Singh、Volodymyr Mnih
- **机构**：DeepMind（业界公认；arXiv 页面未直接列出）
- **arXiv 编号**：2210.14215
- **版本历史**：v1 2022-10-25
- **会议**：ICLR 2023（业界公认；arXiv 页面未直接注明）

## 摘要（据 arXiv 整理）

提出 **Algorithm Distillation (AD)**：通过用因果序列模型对强化学习算法的**训练历史**建模，把 RL 算法蒸馏进神经网络。将 RL 的学习过程视为**跨 episode 的序列预测问题**——在源 RL 算法生成的数据集上训练 Transformer，使其能**在上下文中（in-context）改进策略而无需更新权重**。在稀疏奖励、组合任务结构、像素观测等环境中均有效，且能得到比生成源数据的算法更数据高效的 RL 算法。

## 核心思想

不蒸馏"策略"，而是蒸馏"学习算法本身"：让模型读入"一个 RL 智能体逐步变强的完整学习史"，从而学会"如何学习"，在推理时仅靠上下文就持续自我改进，不改权重。

## 在 Lilian Weng 框架中的位置

属 **Planning → Self-Reflection**：被引为"通过历史轨迹实现上下文内自我改进"的代表，与 Reflexion 的"语言反思 + 记忆"形成不同技术路线的对照。

## 意义

确立了"in-context RL / 把学习算法本身序列化"的范式，是上下文内自我改进、记忆驱动智能体的重要思想来源。

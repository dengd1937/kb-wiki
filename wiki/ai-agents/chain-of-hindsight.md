# Chain of Hindsight：用反馈序列对齐语言模型

> Sources: arXiv 2302.02676（Liu et al., UC Berkeley），2023-02-06；ICLR 2024
> Raw: [Chain of Hindsight（要点/中文）](../../raw/ai-agents/2023-02-06-chain-of-hindsight.md)

## 概述

CoH（2023 年 2 月，UC Berkeley，Pieter Abbeel 组）把**所有反馈转化为句子序列**用于微调模型。模型以"成对的输出 + 反馈"为条件，学会根据反馈生成更好输出并主动纠正负面属性。在摘要、对话基准上显著优于已有对齐方法，是绕开 RLHF 复杂性的轻量路线。

## 核心思想

把"做得好/做得差 + 对应反馈"拼成"后见之明"序列让模型一起读，学会在反馈条件下自我纠正。

## 在 Lilian Weng 框架中的位置

**Planning → Self-Reflection**：作为"从反馈中学习改进"的代表，与 ReAct/Reflexion 并列。

## 意义

提供"用语言序列直接编码反馈来改进模型"的轻量路线，影响后续从反馈学习与自我改进研究。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — 其 Self-Reflection 的代表方法之一。
- [Reflexion](reflexion.md) — 同属自我改进方向（语言反馈 + 记忆）。

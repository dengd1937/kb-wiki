# Chain-of-Thought：思维链提示激发 LLM 推理

> Sources: arXiv 2201.11903（Wei et al., Google Brain），2022-01-28；NeurIPS 2022
> Raw: [Chain-of-Thought（要点/中文）](../../raw/ai-agents/2022-01-28-chain-of-thought.md)

## 概述

Google Brain 团队 2022 年 1 月提出 Chain-of-Thought (CoT) prompting：在 few-shot 示例中提供"带中间推理步骤的演示"，引导模型在给出答案前显式生成推理过程。纯推理时、零训练成本，推理能力随模型规模涌现；540B 模型借此在 GSM8K 取得 SOTA。

## 核心思想

把示例从"问题 → 答案"改为"问题 → 中间推理步骤 → 答案"，触发模型"think step by step"，把难任务拆为可处理的中间步骤。

## 在 Lilian Weng 框架中的位置

**Planning → Task Decomposition** 的基础方法，是 ToT、ReAct 等后续工作的共同起点。

## 意义

LLM 推理与 Agent 规划能力的奠基性提示范式，催生 Zero-shot-CoT、Self-Consistency、ToT 等系列工作。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — CoT 是其 Planning/Task Decomposition 的核心方法。
- [Tree of Thoughts](tree-of-thoughts.md) — CoT 的树状扩展。
- [ReAct](react-reasoning-acting.md) — 在 CoT 推理上叠加行动。

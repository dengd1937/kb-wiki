# LLM+P：用经典规划器赋能 LLM 的最优规划

> Sources: arXiv 2304.11477（Liu et al., UT Austin & SUNY Binghamton），2023-04-22
> Raw: [LLM+P（要点/中文）](../../raw/ai-agents/2023-04-22-llm-p.md)

## 概述

针对"LLM 无法可靠求解长程规划问题"的痛点，LLM+P（2023 年 4 月）让 LLM 把自然语言问题转成 **PDDL**，交由**经典规划器**求最优解，再翻译回自然语言。多数基准上能得最优解，而纯 LLM 对大多数问题给不出可行计划。

## 核心思想

"LLM 做翻译，经典规划器做规划"——典型神经-符号混合：把求解外包给可保证最优性的符号规划器。

## 在 Lilian Weng 框架中的位置

**Planning → Task Decomposition**：代表"借助外部经典规划器"的路线，与 CoT/ToT 的纯 LLM 内部推理形成对照。

## 意义

示范用外部符号工具弥补 LLM 长程规划短板，是 Agent 规划走向神经-符号混合的代表。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — 其 Planning 中"外接规划器"路线的代表。
- [Tree of Thoughts](tree-of-thoughts.md) / [Chain-of-Thought](chain-of-thought.md) — 纯 LLM 内部推理的对照路线。

# Tree of Thoughts：审议式问题求解

> Sources: arXiv 2305.10601（Yao et al., 普林斯顿 & Google DeepMind），2023-05-17；NeurIPS 2023
> Raw: [Tree of Thoughts（要点/中文）](../../raw/ai-agents/2023-05-17-tree-of-thoughts.md)

## 概述

ToT 由 ReAct 同一研究组（Shunyu Yao 等）于 2023 年 5 月提出。在 CoT 之上把推理组织为**树**：每个节点是一个"想法"，可生成多个候选分支，用 LLM 自评估打分，配合 BFS/DFS 搜索与前瞻/回溯做"深思熟虑"的决策。Game of 24 成功率 74%，远超标准提示 GPT-4 的 4%。

## 核心思想

链 → 树：把解题从"一次性线性生成"变为"可搜索、可回退的审议过程"，把经典树搜索与 LLM 推理结合。

## 在 Lilian Weng 框架中的位置

**Planning → Task Decomposition**，CoT 的直接扩展，提供多路径探索 + 自评估。

## 意义

Agent 规划从"线性思维链"走向"审议式搜索"的代表工作。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — 其 Planning/Task Decomposition 的代表方法。
- [Chain-of-Thought](chain-of-thought.md) — ToT 的前身。
- [ReAct](react-reasoning-acting.md) — 同组并行分支（推理-行动方向）。

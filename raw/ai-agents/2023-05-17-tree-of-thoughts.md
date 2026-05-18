# Tree of Thoughts: Deliberate Problem Solving with Large Language Models

> Source: https://arxiv.org/abs/2305.10601
> Collected: 2026-05-18
> Published: 2023-05-17（arXiv v1；v2 2023-12-03 NeurIPS camera-ready）；NeurIPS 2023

## 元数据

- **标题**：Tree of Thoughts: Deliberate Problem Solving with Large Language Models
- **作者**：Shunyu Yao、Dian Yu、Jeffrey Zhao、Izhak Shafran、Thomas L. Griffiths、Yuan Cao、Karthik Narasimhan
- **机构**：普林斯顿大学 & Google DeepMind（与 ReAct 同一研究组，Shunyu Yao / Karthik Narasimhan / Yuan Cao 重叠）
- **arXiv 编号**：2305.10601
- **版本历史**：v1 2023-05-17；v2 2023-12-03
- **会议**：NeurIPS 2023

## 摘要（据 arXiv 整理）

ToT 在 Chain-of-Thought 之上扩展：让模型在连贯的文本单元（thoughts，作为解题的中间步骤）上进行探索。模型可以**考虑多条不同推理路径、对选择做自我评估**，并支持前瞻（lookahead）与回溯（backtracking），从而进行"深思熟虑"的决策。结果显著：在 Game of 24 上成功率达 74%，而标准提示的 GPT-4 仅 4%。

## 核心思想

把推理过程组织为**树**而非链：每个节点是一个"想法"，可生成多个候选分支；用 LLM 自评估给分，配合 BFS/DFS 搜索做前瞻与回溯。将解题从"一次性线性生成"变为"可搜索、可回退的审议过程"。

## 在 Lilian Weng 框架中的位置

属 **Planning → Task Decomposition**：是 CoT 的直接扩展（链 → 树），提供多路径探索与自评估能力。

## 意义

把经典搜索（树搜索、回溯）与 LLM 推理结合，显著提升需要规划/搜索的硬任务表现，是 Agent 规划能力从"线性思维链"走向"审议式搜索"的代表工作。

## 相关
- CoT（arXiv 2201.11903）：ToT 的前身（链 → 树）。
- ReAct（arXiv 2210.03629）：同组工作，推理-行动方向的并行分支。

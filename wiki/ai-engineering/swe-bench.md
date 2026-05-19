# SWE-bench：真实 GitHub issue 的编码 Agent 评测基准

> Sources: arXiv 2310.06770（Jimenez et al., 普林斯顿等），2023-10-10；ICLR 2024
> Raw: [SWE-bench（verbatim/中文）](../../raw/ai-engineering/2023-10-10-swe-bench.md)

## 概述

普林斯顿团队（与 ReAct/ToT/Reflexion 同组）2023 年 10 月提出。SWE-bench 含 **2,294 个**取自 12 个流行 Python 仓库真实 GitHub issue→PR 的软件工程问题：给定代码库与 issue 描述，模型需编辑代码库修复。需跨函数/类/文件协调、与执行环境交互、处理超长上下文。初始评测中最强的 Claude 2 仅解决 **1.96%**。

## 核心贡献

- 首个**真实 issue→PR**、大规模、可持续、贴近生产的编码 Agent 基准。
- 用"能否真正修好真实 bug"取代玩具式代码生成，**定义了 Coding Agent 时代的标尺**。
- 揭示巨大差距（SOTA≈2%），驱动此后整个 Coding Agent 竞赛。

## 在工程化线中的位置

**能力跃迁**阶段的"标尺/起点"——把 Coding Agent 从概念变成可量化目标。出自 ReAct 同一研究组。

## 相关条目

- [2024–25 能力跃迁](capability-leap.md) — 本阶段综述。
- [SWE-agent](swe-agent.md) — 在本基准上以 ACI 取得突破的同组方法。
- [ReAct](../ai-agents/react-reasoning-acting.md) — 同一普林斯顿研究组谱系。

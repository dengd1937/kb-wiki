# Building Effective Agents：回归克制的 Agent 设计模式

> Sources: Anthropic Engineering《Building Effective Agents》，2024-12-19
> Raw: [原文要点 / verbatim](../../raw/ai-engineering/2024-12-19-building-effective-agents.md)

## 概述

Anthropic 2024-12-19 的工程指南。核心：最成功的 agent 用**简单、可组合的模式**，而非复杂框架；从最简方案起步，仅当可度量收益证明值得才加复杂度。

> "It's about building the *right* system for your needs."

## Workflows vs Agents

- **Workflow**：LLM/工具走**预定义代码路径**——用于明确、可预测、需一致性的任务。
- **Agent**：LLM **动态自主**决定流程与工具——用于步数不可预测、需模型决策的开放问题。

基础构件：**Augmented LLM**（LLM + 检索 + 工具 + 记忆）。

## 五种工作流模式

| 模式 | 适用 |
|---|---|
| Prompt Chaining | 可分解的顺序任务 |
| Routing | 需分类分流的不同类别 |
| Parallelization | 独立子任务(sectioning)/多次取共识(voting) |
| Orchestrator-Workers | 子任务不可预测，动态拆解委派 |
| Evaluator-Optimizer | 有明确评判标准的迭代优化 |

## 三条核心原则

1. 设计**简单性**；2. 显式规划步骤保证**透明性**；3. 精心打造**ACI（智能体-计算机接口）**。

## 在工程化线中的位置

**设计模式收敛**节点：框架浪潮（过度复杂）+ 协议标准化（MCP/A2A）之后，本文回归克制，遏制过度工程。第 3 原则「精心设计 ACI」直接呼应 SWE-agent，并通往 Harness「为智能体设计环境」的生产范式——是通往 Harness 的最后一块拼图。

## 相关条目

- [2023–24 Agent 框架浪潮](agent-framework-wave.md) — 本文所回应的过度复杂阶段。
- [MCP](model-context-protocol.md) / [A2A](a2a-protocol.md) — 同期协议标准化（接入层）。
- [SWE-agent](swe-agent.md) — 第 3 原则 ACI 思想的来源。
- [Harness Engineering](harness-engineering.md) — 本节点直接通往的下游生产范式（终点）。

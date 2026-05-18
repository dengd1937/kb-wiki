# ReAct：推理与行动协同的语言模型范式

> Sources: arXiv 2210.03629（Yao et al., 普林斯顿 & Google Research），2022-10-06；ICLR 2023
> Raw: [ReAct（原文/中文）](../../raw/ai-agents/2022-10-06-react-reasoning-acting.md)

## 概述

ReAct 由普林斯顿与 Google Research 于 2022 年 10 月提出，是现代 LLM Agent 的奠基性工作之一。核心贡献：让大语言模型以**交错方式**同时生成"推理轨迹（Reasoning）"与"具体行动（Acting）"，使二者协同——推理帮助制定/修订计划并处理异常，行动则让模型与外部知识库或环境交互以接地（grounding）。该方法仅用 1–2 个上下文示例即在多类任务上超越当时 SOTA，并显著提升可解释性与可信度。

## 核心机制：Thought–Action–Observation 循环

ReAct 在同一推理循环内交错产生三类内容，直到得到最终答案：

- **Thought（思考）**：自然语言推理，分解任务、制定与修订计划、处理异常。
- **Action（行动）**：调用外部工具/环境的结构化动作，如 `search[entity]`、`lookup[string]`、`finish[answer]`。
- **Observation（观察）**：环境返回结果，反馈进上下文驱动下一步思考。

循环形式：`Thought → Action → Observation → Thought → …`

## 它解决了什么

- **纯思维链（CoT）**：缺乏外部接地，易幻觉、错误沿推理链传播、事实不可核验。
- **纯行动（Act-only）**：能交互但缺高层推理，难分解复杂目标、难从失败恢复。
- **ReAct 协同**：推理为行动提供计划与纠错，行动为推理提供真实接地，二者互补。

## 关键结果

| 任务类型 | 基准 | 结果 |
|---|---|---|
| 知识密集型推理 | HotpotQA / FEVER | 经 Wikipedia API 接地，显著缓解幻觉与错误传播 |
| 交互式决策 | ALFWorld | 绝对成功率 **+34%**（vs 模仿/强化学习） |
| 交互式决策 | WebShop | 绝对成功率 **+10%** |

- 仅需 **1–2 个 few-shot 示例**，无需微调。
- 生成轨迹**类人、可解释、可信**，便于人类诊断与干预。

## 历史意义

ReAct 早于 ChatGPT 发布，首次系统性地把"推理 + 工具调用 + 环境反馈"统一进一个提示驱动的循环，奠定了后续 Agent 范式（工具使用、ReAct-style agent loop、LangChain Agent、AutoGPT 等）的基础。当下（2026）热门的 Agent Harness 等工程范式，其智能体内核仍可追溯至这一 Thought–Action–Observation 循环。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — Lilian Weng 的统一框架，ReAct 属其 Planning 组件的核心方法。
- [Harness Engineering: Building Software with AI Agents](../ai-engineering/harness-engineering.md) — ReAct 式 agent loop 在工程化落地的当代延伸。

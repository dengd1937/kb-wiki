# 2024–25 能力跃迁：Computer Use / Coding Agent

> Sources: 综合综述（SWE-bench / SWE-agent / Devin / Anthropic Computer Use / OpenAI Operator 各官方出处），时间线已交叉核对
> Raw: [来源与时间线明细](../../raw/ai-engineering/2024-03-12-capability-leap.md)

## 概述

框架浪潮解决"如何编排"后，2024 年 Agent 发生能力质变：从"调 API + 编排循环"跃升到**端到端写代码**与**直接操作计算机（GUI/浏览器）**。位于 Function Calling 铰链之后、MCP 标准化之前。

## 两条子线时间线

**A. Coding Agent**

| 时间 | 节点 | 主体 |
|---|---|---|
| 2023-10 | SWE-bench（评测标尺，SOTA≈1.96%）| 普林斯顿 |
| 2024-03 | Devin（"首个全自主 AI 软件工程师"）| Cognition |
| 2024-05 | SWE-agent（ACI，SWE-bench→12.5%）| 普林斯顿 |
| 2025+ | Claude Code / OpenAI Codex（生产级）| Anthropic / OpenAI |

**B. Computer Use**

| 时间 | 节点 | 主体 |
|---|---|---|
| 2024-10-22 | Anthropic Computer Use（首个通用计算机操作前沿模型）| Anthropic |
| 2025-01-23 | OpenAI Operator（浏览器自主操作，CUA）| OpenAI |

## 主线脉络

标尺先行（SWE-bench）→ 产品引爆（Devin）→ 方法突破（SWE-agent 的 ACI）→ 跨出代码操作任意 GUI（Computer Use / Operator）。

## 意义与下游

能力跃迁让 Agent 能自主交付，但也使**工具/上下文接入碎片化**不可忍受——直接催生 **MCP（2024-11）** 标准化、**《Building Effective Agents》（2024-12）** 设计模式收敛，最终汇入 **Harness Engineering（2026）**。SWE-agent 的 ACI 是 Harness「为智能体设计环境」的直接前驱。

## 相关条目

- [2023–24 Agent 框架浪潮](agent-framework-wave.md) — 上一阶段（如何编排）。
- [SWE-bench](swe-bench.md) / [SWE-agent](swe-agent.md) — 本阶段 Coding Agent 关键论文。
- [OpenAI Function Calling](../ai-agents/openai-function-calling.md) — 上游铰链。
- [MCP: Model Context Protocol](model-context-protocol.md) — 本阶段碎片化痛点直接催生的下游标准化。
- [Harness Engineering](harness-engineering.md) — 下游生产范式（终点）。

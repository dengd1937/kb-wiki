# 2023–24 Agent 框架浪潮

> Sources: 综合综述（LangChain / AutoGPT / BabyAGI / AutoGen / CrewAI / LangGraph 各官方出处），时间线已交叉核对
> Raw: [来源与时间线明细](../../raw/ai-engineering/2023-03-30-agent-framework-wave.md)

## 概述

Function Calling（2023-06）让单次工具调用可靠后，工程重心转向"把调用编排成自主循环/多智能体协作"。2023 下半年至 2024 涌现一整波 Agent 框架，是工程化线的第一阶段，衔接铰链节点 Function Calling 与后续标准化/收敛。

## 时间线

| 时间 | 框架 | 出品 | 范式 |
|---|---|---|---|
| 2022-10 | LangChain | Harrison Chase | 组件化编排，2023 成熟 Agent 抽象 |
| 2023-03-30 | AutoGPT | Significant Gravitas | 自主目标驱动循环，引爆认知 |
| 2023-04 | BabyAGI | Yohei Nakajima | 极简任务规划+队列自主 agent |
| 2023-08 | AutoGen | 微软研究院 | 多智能体对话协作 |
| 2023-11 | CrewAI | João Moura | 角色分工多智能体团队 |
| 2024-01 | LangGraph | LangChain 团队 | 图/状态机有状态编排 |

## 四段脉络

1. **自主派狂热与祛魅**（AutoGPT/BabyAGI）：引爆认知，但跑飞/不收敛/成本失控，证明纯自主尚不可靠。
2. **编排库沉淀**（LangChain）：模块化，成事实生态。
3. **多智能体协作**（AutoGen / CrewAI）：单 agent → 多 agent 分工。
4. **有状态/图编排**（LangGraph）：图与状态机取代脆弱隐式循环，走向可控可恢复。

## 意义与下游

把原子能力组织成可用系统，但带来**碎片化 + 过度复杂**——分别催生 MCP（标准化接入）与《Building Effective Agents》（设计模式收敛），最终汇入 Harness Engineering 生产范式。

## 相关条目

- [OpenAI Function Calling](../ai-agents/openai-function-calling.md) — 框架浪潮的前提（可靠工具调用）。
- [AutoGen: Multi-Agent Conversation](autogen.md) — 本浪潮中多智能体协作的代表（单独条目）。
- [2024–25 能力跃迁](capability-leap.md) — 本浪潮之后的下一阶段（能力质变）。
- [Harness Engineering: Building Software with AI Agents](harness-engineering.md) — 工程化线终点。

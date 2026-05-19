# 2023–24 Agent 框架浪潮（综述）

> 性质: 综合综述条目（无单一来源，下列为各节点权威出处）
> Collected: 2026-05-18
> 时间跨度: 2022-10 ～ 2024（锚点 2023-03-30 AutoGPT 发布，框架爆发起点）

## 来源（各节点）

- LangChain — https://en.wikipedia.org/wiki/LangChain ；blog.langchain.com
- AutoGPT — https://en.wikipedia.org/wiki/AutoGPT ；github.com/Significant-Gravitas/AutoGPT
- BabyAGI — https://github.com/yoheinakajima/babyagi ；yoheinakajima.com/birth-of-babyagi/
- AutoGen — arXiv 2308.08155（见单独条目）
- CrewAI — github.com/crewAIInc/crewAI ；docs.crewai.com
- LangGraph — changelog.langchain.com

## 背景

Function Calling（2023-06）让单次工具调用变得可靠后，工程重心迅速转向：**如何把"调用"编排成"自主循环 / 多智能体协作"**。2023 下半年至 2024 出现一整波 Agent 框架，构成工程化线的第一阶段。

## 时间线与节点（已交叉核对）

| 时间 | 框架 | 作者/出品 | 范式 |
|---|---|---|---|
| 2022-10 | **LangChain** | Harrison Chase（后成立 LangChain 公司） | 链式/组件化编排；2023 年成熟 Agent 抽象，Q3 引入 LCEL |
| 2023-03-30 | **AutoGPT** | Toran Bruce Richards / Significant Gravitas | 自主目标驱动循环；引爆大众认知 |
| 2023-04 | **BabyAGI** | Yohei Nakajima（3 月创建，4 月走红） | 任务规划 + 任务队列的极简自主 agent |
| 2023-08 | **AutoGen** | 微软研究院 | 多智能体对话协作（见单独条目） |
| 2023-11 | **CrewAI** | João Moura（AI Fund 孵化） | 角色分工的多智能体团队 |
| 2024-01 | **LangGraph** | LangChain 团队 | 图/状态机式有状态编排（后成 LangChain 默认运行时） |

## 主线脉络

1. **自主派狂热与祛魅**（AutoGPT/BabyAGI，2023 春）：承诺"给目标自动跑完"，引爆认知，但很快暴露跑飞、不收敛、成本失控——证明纯自主链条尚不可靠。
2. **编排库沉淀**（LangChain，2023）：把 prompt/工具/记忆/agent 模块化，成为事实生态。
3. **多智能体协作**（AutoGen 2023 / CrewAI 2023-11）：从单 agent 转向"多 agent 分工对话"。
4. **有状态/图编排**（LangGraph 2024）：用图与状态机取代脆弱的隐式循环，走向可控、可恢复。

## 意义与下游

框架浪潮把 Function Calling 的原子能力组织成可用系统，但也带来碎片化（每个框架一套工具胶水、抽象不一）与过度复杂。这两点分别催生：

- **MCP（2024-11）**：标准化工具/上下文接入。
- **《Building Effective Agents》（Anthropic, 2024-12）**：回归克制，区分 workflow 与 agent 的设计模式收敛。
- 最终汇入 **Harness Engineering（2026）** 的生产范式。
